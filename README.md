# Deed: A Decentralised Exchange for Internet Assets

Deed is a decentralized exchange for internet assets with a progressive web app frontend where users can tokenise any internet asset to discover its true & fair market value while generating passive income from trade fees. Traders can discover & trade the next 100x gem & earn ownership of the exchange through DEED tokens granted based on monthly trading volumes. DEED is a utility token of Deed DEX that grants governance rights to users who can also stake DEED to receive a share of DEX earnings. Being fully decentralized 100% earnings of DEX are distributed to its users.

## Introduction

More than 40% of internet bandwidth is attributed to Big Tech (Meta, Alphabet, Apple, Amazon & Microsoft) that hosts millions of assets & applications. Big Tech has solved mass content distribution at scale while being highly centralized institutions to accrue a market capitalization of  over $10 trillion over the last few decades. Even though Big Tech has accrued trillions in market cap yet there is no way to directly attribute a piece of this value to the assets owned by millions of creators & entrepreneurs hosting assets on Big Tech platforms. Whether a user owns a Youtube channel or an Amazon Seller account or a Saas Web app  or a Game App on an iOS store or just a simple website, they have no way of discovering the true market values of their assets. Some of these assets might be generating millions in cash flows for their owners, but still there is no way to discover their value. Traditional finance is expensive & inefficient to solve this & hence we need a DeFi solution.


What is needed is a decentralized exchange for tokenizing & trading of tokens connected to internet assets to discover their true & fair market value on a standalone basis. This requires mechanisms to validate , tokenize & trade tokens representing such internet assets in a way that each token is uniquely attributed to their underlying internet asset on-chain. Mechanisms are also required to onboard asset data (subs, users, followers, earnings, gross revenues etc) on-chain for token traders to make trading decisions about asset valuation & yield sharing systems that pass earnings of a real asset to their on-chain token holders. At its core, the decentralized exchange contract is powered by trade pools (aka "Markets") with incentives defined in contract for both asset owners & traders. The exchange would be owned, operated & governed by users with incentives & governance (control) tied to their contributions to maintain full decentralization.



## Design

### Problems


1. Tokenization is complex with multiple barriers around paying gas fees & authorising individual transactions.
2. Launching liquidity pools (trade pools) is neither easy nor intitutive even for many crypto natives.
3. Tokenisation & trading of tokens happens on separate apps, causing switching pain.
4. Tokensation & liquidity pools are disconnected from real world assets.
5. Mobile based tokenisation & trading has better UX, but crypto apps mostly get rejected on app stores.
6. Rug & pull scams are common on popular AMMs with no safeguards for traders.

### Goal

To build Deed as a Progressive web app that makes the experience of tokenization & trading of tokens connected to internet assets easy, intuitive & safe.

### Prototype

The prototype is built on Solana using Tatum API infra to validate the thesis of tokenisation & tradings of tokens connected to internet assets. Instead of building the full fledged DEX ourright, the prototype enables an internet user to tokenise internet assets & then launch markets for such tokens paired with the utility token DEED on an Orderbook powered by Tatum. While Tatum itself is not decentralised, the objective of the prototype is to establish some product market fit in terms of trading volumes and seed an intial user base. Once base metrics have been established, Tatum infra would be phased out with decentralised infra compeletely. 
Tokenisation & trading are live on prototype version-1 at https://deed.so/download-app



## Trade Pools (aka "Markets")

Trade pools are smart contracts that hold an equal amount of tokens (connected to an internet asset) & another cryptocurrency native to chain for facilitating exchange & trade utilities. [Solana token swap program](https://spl.solana.com/token-swap) has been used with some customizations around liquidity lock mechanisms & bonding curve to create token swap pools of assets tokenised on Solana. Solana is one of the lowest gas fees L1 chain. This enables Deed App to provide wallets that can have unlimited transactions at ZERO gas fee where the gas cost of any transaction on Deed app is borne by a central wallet of the app & not the user thus making transactions gas free for the user improving UX. Users can seamlessly trade through embedded wallets with 0 gas fees.

### Trade fee

Asset owners can set custom trade fees for every trade pool in the range of 1%-10%, based on projected trading volumes & estimated earnings they would like to attain. Custom trade fees opens up use cases for multiple type of assets where owners might want to keep a low range expecting high volumes & vice-versa. 

Function to set custom trade fee for each trade pool

```bash

// Define the custom trade fee

let tradeFee = 0.05; // Default trade fee is 5%

// Function to set a custom trade fee

function setTradeFee(customFee) {
    if (customFee >= 0.01 && customFee <= 0.1) {
        tradeFee = customFee;
    } else {
        throw new Error('Trade fee must be a value between 1% and 10%');
    }
}

```

### Locking liquidity

The purpose of trade pools is to help the market determine value of an internet assets by inducing liquidity in an otherwise illiquid asset & hence the entire supply linked to tokens represented by internet assets is locked in the trade pool at launch. Pool locking mechansim is experimental with time based locking that will lock the pool for a defined amount of time. Lock tenures help establish stability & provide guarantees to traders that token creators can't just wind up a token anytime & rug pull thier funds.

```bash

/ Define the pool lock status and duration

let isPoolLocked = false;
let poolLockDuration = 0;

// Function to lock the pool for a specific period

async function lockPool(duration) {
    isPoolLocked = true;
    poolLockDuration = duration;
    setTimeout(() => {
        isPoolLocked = false;
        poolLockDuration = 0;
    }, duration);
}

// Function to withdraw liquidity when the pool is unlocked
async function withdrawLiquidity(userPublicKey, poolTokenAmount) {
    try {
        if (!isPoolLocked) {
            const userTokenBalance = await checkUserTokenBalance(userPublicKey, poolTokenMint);
            if (userTokenBalance >= poolTokenAmount) {
                await transferTokens(userPublicKey, poolTokenMint, poolTokenAmount, poolTokenAccount);
                const tokenAAmount = calculateTokenAAmount(poolTokenAmount);
                const tokenBAmount = calculateTokenBAmount(poolTokenAmount);
                await transferTokens(poolTokenAccount, tokenA, tokenAAmount, userPublicKey);
                await transferTokens(poolTokenAccount, tokenB, tokenBAmount, userPublicKey);
            } else {
                throw new Error('Insufficient pool token balance');
            }
        } else {
            throw new Error('Cannot withdraw liquidity. Pool is currently locked');
        }
    } catch (error) {
        console.error("Error withdrawing liquidity:", error);
    }
}

// Function to deposit liquidity
async function depositLiquidity(userPublicKey, tokenAAmount, tokenBAmount) {
    try {
        if (!isPoolLocked) {
            const tokenAAmountToDeposit = calculateTokenAAmountToDeposit(tokenAAmount);
            const tokenBAmountToDeposit = calculateTokenBAmountToDeposit(tokenBAmount);
            await transferTokens(userPublicKey, tokenA, tokenAAmountToDeposit, poolTokenAccount);
            await transferTokens(userPublicKey, tokenB, tokenBAmountToDeposit, poolTokenAccount);
            const poolTokenAmountToMint = calculatePoolTokenAmount(tokenAAmountToDeposit, tokenBAmountToDeposit);
            await mintPoolTokens(userPublicKey, poolTokenMint, poolTokenAccount, poolTokenAmountToMint);
        } else {
            throw new Error('Cannot deposit liquidity. Pool is currently locked');
        }
    } catch (error) {
        console.error("Error depositing liquidity:", error);
    }
}
```


### Gas-free

Any transaction done on Trade pools would be gas-free for enhanced user experience & hence L1/ L2 blockchains with ultra low gas fees only would be suitable for Deed DEX contract in comparison to Ethereum where a single swap transaction can cost upwards of $20. All gas costs would be incurred by a central wallet of Deed to enhance user experience.



### Bonding curve

Trade pools on Deed DEX use Constant Product Market Maker (CPMM) bonding curves with bidding systems to initialize liquidity on the exchange. The problem with traditional AMMs using constant product curves is high volatility at low liquidity making it easier for users with low liquidity to create rug pulls. This can lead to absurd market caps that are disconnected from the true value of underlying real assets. To take an example let's say a user creates a pool with 0.01 ETH ($20) by locking a total supply of 1 million tokens of REAL_ASSET thereby setting the price of REAL_ASSET at $0.00001. Now if the next buy transaction in the pool is for 1 ETH ($2000), the price of REAL_ASSET rises to $0.20402 which is a 1020000% increase from initial price while total liquidity is only $2020. However if the initial liquidity in this example is enhanced to 10 ETH ($20,000), then the price of token REAL_ASSET rises by only 20% on the next transaction. One can say that even when initial liquidity is $20,000, if the next transaction introduces 1000 ETH ($2,000,000) of liquidity, then the price change is again absurd, but we must remember that finding higher liquidity acts as a key barrier in doing so & our goal is to enhance such barriers to ensure efficient asset markets.   

### Dutch auctions

The solution to higher initial liquidity is a dutch auction mechanism in-built into the DEX trade pool contract. Trade pool contract enables asset owners to launch Dutch auctions for total supply of tokens by setting the initial market value (same as initial liquidity), period of auction, price reduction logic based on demand & trade fees charged on each buy/sell transaction. Dutch auctions have a quite successful track record at discovering optimum market valuation of real assets. Traders lock funds into a trade pool contract to bid for token quantity at a given price which is reduced at defined intervals. Traders can update bids till closing time & in case Dutch price auction is not successful at the end of auction period or if final clearing price of tokens is not accepted by asset owner, the auction is nulled & funds deposited into trade pool contract can be withdrawn.


### Asset data

In various Tradifi asset markets, trades are based on changes in asset fundamentals or technical patterns discovered by traders through broker terminals or news coverage. In Defi, without providing underlying real asset data to traders, most would be speculating rather than taking calculated decisions on token prices. Hence real asset data points like gross revenues, users, subscribers, patrons, churn, followers, ad earnings etc based on asset type are available on Deed DEX app to traders for making informed decisions regarding valuation of assets subject to approval of asset owners who can control release of asset data based on a set frequency or can approve real time data feeds based on their preferences for sharing asset data. The quantum & frequency of asset data shared can also help traders determine quality of an asset which would ultimately reflect in its valuation. 



### Asset validation 


There are more than 2 million token contracts that have been created so far, however most lack utility as they are not connected with a real world asset. Deed enables not only linking of a real asset to its tokenized version on-chain through asset validation but will offer the entire underlying infra for developers to connect trusted oracles & tokenize multitude of real assets with their own frontend applications.


Real assets (or internet assets) are connected with their on-chain contracts through zero knowledge (ZK) proofs validating asset ownership, location & condition. In absence of ZK proofs , any user can claim to be owner of  a real asset by deploying contracts with token symbols similar to original tokens. Adding metadata to contracts doesn’t solve the problem as they can be duplicated with no validation proof. Asset metadata (ownership, location, condition etc) can be validated through user authentication with trusted third party oracles that host or store such assets which can then be used to create zero knowledge proofs without revealing the actual owner or asset details. Simple cryptographic signatures can also be used to validate asset metadata where users wouldn’t want to protect privacy like in many public internet assets, however we leave the final choice with users by providing both alternatives. Any user can claim ownership to a tokenised version of an internet asset on Deed through asset validation mechanism. So for instance a user can tokenise a X account belonging to Elon Musk, but only Elon Musk can validate & claim ownership of the tokens connected to his handle through asset validation. The user who tokenised, will continue to enjoy any trade fees generated till the time an asset is claimed, post which all tokens and fees connected to such tokens would move to validators wallet.




## Full Decentralization

In order to ensure continuity of Deed DEX it must be fully owned, operated & governed by its users in the long run. However most decentralized exchanges often claim but are not fully decentralized. This is evident in the manner how their earnings & ownership are distributed amongst users. For any exchange contract to be fully decentralized it should distribute 100% of its earnings to its users along with a path to progressively pass major ownership of the exchange to users based on defined metrics. Along with decentralization it is essential that users are incentivized enough to bootstrap & grow liquidity on Deed DEX for an indefinite period of time. 


### Earnings distribution

To incentivise creation of trade pools,  asset owners tokenizing real assets can set a trade fee during the tokenization process that they would receive on each buy or sell transaction from the trade pool created . Trade fee can be set between 1-10% initially based on the type of asset being tokenized & the fee range is open for revision based on voting by users of Deed DEX through DAO governance proposals.  Asset owners receive 70% of the trade fee set (to attract quality assets) while 30% is added to the stake rewards pool for distribution to traders. Thus 100% of the earnings of Deed DEX are distributed to its users. To pursue progressive decentralization asset owners have the option to receive  trade fee earnings in any native currency or stablecoin with which the trade pool has been created with an option to convert their earnings into exchange tokens (DEED) for ownership & staking rewards. 


### Ownership distribution

The path to full decentralization begins as a variation of progressive decentralization whereby Deed DEX ownership is not allocated but rather earned by users & developers alike. Users earn DEED based on trade volumes or by converting trade fee earnings while developers are allocated DEED based on DEX development milestones. 

DEED has a fixed total supply of 1.24 billion tokens that can be earned by trading on DEX or by contributing towards development of the exchange. 1 billion (80.65% of total supply) tokens are reserved for users tokenising & trading on DEX while 0.24 billion tokens (19.35% of total supply) are reserved for developers to be allocated over a period of time. Users earn 1 DEED for every $100 traded on DEX, which implies that 1 billion DEEDs would have been earned & distributed when Deed DEX attains a cumulative $100 billion of trading volume. Developers are allocated DEED to compensate for the time & effort contributed towards DEX development at the rate of 0.6% of total supply available for users (6M tokens monthly) over a period of 40 months which is expected time for DEX to hit $100 billion in trading volume & completion of full decentralization phase. All developer tokens would be locked for 6-12 months from the date of listing DEED on any major CEX or DEX or from the date of allocation whichever is later. 


## Deed DAO

All users of Deed DEX holding DEED shall be default members of Deed DAO with 1 voting per DEED held as on date. Deed DAO would be the primary governance body consisting of DEX users & developers with powers to vote on any changes in DEX contract as defined in contract upon creation. While its important to have equal opportunity for every user (hance 1 vote for 1 DEED), however the voting system design would encourage participation & provide control to most active users as they earn DEED based on activity on the DEX. Any DAO member can create a proposal which when ratified by at least 10% of total voting power, would open for voting to all members for 14 days. Any proposal ratified by 51% or more of voting power in circulation at the time of the vote shall have passed, while a proposal ratified by less than 51% voting power would be annulled. Functions (not exhanustive) of Deed DEX that DAO members can vote on-

* Launching Deed DEX contracts on new blockchains
* Approving trusted third party oracle integrations with Deed DEX
* Modification in bidding mechanism
* Setting trade fee range based on various asset types
* Determining DEX earnings distribution ratios between asset owners & traders
* Updating DEED token rewards computations ($1 trading volume= 1 DEED)
* Adding or removing an asset type on exchange 
* Implementation of staking & yield farming contracts
* Listing DEED on other CEXs or DEXs


## Links

1. [Download App](https://deed.so/download-app)
   


