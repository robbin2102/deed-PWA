# Deed: A Decentralised Exchange for Real Assets

Deed is a fully decentralized exchange for real assets with a progressive web app frontend where anyone can tokenise any real assets (internet, commodities, people etc) to discover market valuation of their assets while earning income from trade fees. Traders can earn ownership of the exchange through DEED tokens granted based on monthly trading volumes. DEED is Deed DEX token that can be staked to receive a share of earnings. Being fully decentralized 100% earnings of DEX are distributed to its users who fully own & govern the DEX through Deed DAO.

## Introduction

More than 40% of internet bandwidth is attributed to Big Tech (Meta, Alphabet, Apple, Amazon & Microsoft) that hosts millions of assets & applications. Big Tech has solved mass content distribution at scale while being highly centralized institutions to accrue a market capitalization of  over $10 trillion over the last few decades. Even though Big Tech has accrued trillions in market cap yet there is no way to directly attribute a piece of this value to the assets owned by millions of creators & entrepreneurs hosting assets on Big Tech platforms. Whether a user owns a Youtube channel or an Amazon Seller account or a Saas Web app  or a Game App on an iOS store, they have no way of discovering the true market values of their assets. Some of these assets might be generating millions in cash flows for their owners, but still there is no way to discover their value. Traditional finance is expensive & inefficient to solve this & hence we need a decentralized financial solution.


What is needed is a decentralized exchange for tokenizing & trading of tokens connected to real assets (or internet assets) to discover their true & fair market value on a standalone basis. This requires mechanisms to validate , tokenize & trade tokens representing such real assets (or internet assets) in a way that each token is uniquely attributed to their real asset on-chain. Mechanisms are also required to onboard asset data (subs, users, followers, earnings, gross revenues etc) on-chain for token traders to make trading decisions about asset valuation & yield sharing systems that pass earnings of a real asset to their on-chain token holders. At its core, the decentralized exchange contract is powered by trade pools with incentives defined in contract for both asset owners & traders. The exchange would be owned, operated & governed by users with incentives & governance (control) tied to their contributions to maintain full decentralization.



## Design

### Problems


1. Tokenization is complex with multiple barriers around paying gas fees & authorising individual transactions.
2. Launching liquidity pools (trade pools) is neither easy nor intitutive even for many crypto natives.
3. Tokenisation & trading of tokens happens on separate apps, causing pain to switch.
4. Tokensation & liquidity pools are disconnected from real world assets.
5. Mobile based tokenisation & trading has better UX, but crypto apps mostly get rejected on app stores.
6. Rug & pull scams are common on popular AMMs with no safeguards for traders.

### Goal

To build Deed as a Progressive web app that makes the experience of tokenization & trading of tokens connected to real assets easy, intuitive & safe.

### Prototype (screens)




https://github.com/robbin2102/deed-PWA/assets/103725283/5ce3259c-4224-425d-87ee-4287b071ea3e





See [Prototype](https://deed.so/version-test) for all screens.

## Trade Pools

Trade pools are smart contracts that hold an equal amount of tokens (connected to an internet asset) & another cryptocurrency native to chain for facilitating exchange & trade utilities. [Solana token swap program](https://spl.solana.com/token-swap) has been used as base contract with some customizations around liquidity lock mechanisms & bonding curve to create token swap pools of assets tokenised on Solana. A key advantage of using Solana is that the cost of deploying token contracts & launching trade swap pools is almost 100x lower than other L1 & L2s. This enables Deed App to provide wallets that can have unlimited transactions at ZERO gas fee where the gas cost of any transaction on Deed app is borne by a central wallet of the app & not the user thus making transactions gas free for the user improving UX. Users can seamlessly trade through embedded wallets with 0 gas fees.

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

The purpose of trade pools is to help the market determine value of an internet assets by inducing liquidity in an otherwise illiquid asset & hence the entire supply linked to tokens represented by internet assets is locked in the trade pool at launch. Pool locking mechansim is experimental with time based locking that will lock the pool for a defined amount of time. In version 1.0 locking is for indefinite period of time set for 999 years by default with a provision to open up custom locking durations for future use cases. When a pool is locked owners can only deposit liquidity while withrawals are allowed only when pools get unlocked. Locking liquidity of owners provides a secure mechanism for traders to trade without rug pulls based on supply. 

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

Real assets (or internet assets) are connected with their on-chain contracts through zero knowledge (ZK) proofs validating asset ownership, location & condition. In absence of ZK proofs , any user can claim to be owner of  a real asset by deploying contracts with token symbols similar to original tokens. Adding metadata to contracts doesn’t solve the problem as they can be duplicated with no validation proof. Asset metadata (ownership, location, condition etc) can be validated through user authentication with trusted third party oracles that host or store such assets which can then be used to create zero knowledge proofs without revealing the actual owner or asset details. Simple cryptographic signatures can also be used to validate asset metadata where users wouldn’t want to protect privacy like in many public internet assets, however we leave the final choice with users by providing both alternatives. 

There are more than 2 million token contracts that have been created so far, however most lack utility as they are not connected with a real asset or a blockchain network. Deed enables not only linking of a real asset to its tokenized version on-chain through asset validation but will offer the entire underlying infra for developers to connect trusted oracles & tokenize multitude of real assets with their own frontend applications.


## Full Decentralization

In order to ensure continuity of Deed DEX it must be fully owned, operated & governed by its users in the long run. However most decentralized exchanges often claim but are not fully decentralized. This is evident in the manner how their earnings & ownership are distributed amongst users. For any exchange contract to be fully decentralized it should distribute 100% of its earnings to its users along with a path to progressively pass major ownership of the exchange to users based on defined metrics. Along with decentralization it is essential that users are incentivized enough to bootstrap & grow liquidity on Deed DEX. 


### Earnings distribution

To incentivise creation of trade pools,  asset owners tokenizing real assets can set a trade fee during the tokenization process that they would receive on each buy or sell transaction from the trade pool created . Trade fee can be set between 1-10% initially based on the type of asset being tokenized & the fee range is open for revision based on voting by users of Deed DEX through DAO governance proposals.  Asset owners receive a higher 60% of the trade fee set (to attract quality assets) while 40% is added to the stake rewards pool for distribution to traders. Thus 100% of the earnings of Deed DEX are distributed to its users. To pursue progressive decentralization asset owners have the option to receive  trade fee earnings in any native currency or stablecoin with which the trade pool has been created with an option to convert their earnings into exchange tokens (DEED) for ownership & staking rewards. By converting trade fee earnings to DEED, asset owners can stake to get a share of the remaining 40% of DEX earnings, thus enhancing overall yields while securing ownership of the exchange. 



### Ownership distribution

The path to full decentralization begins as a variation of progressive decentralization  whereby Deed DEX ownership is not allocated but rather earned by users & developers alike without any pre-mine or pre-mint of exchange token (DEED). Users earn DEED based on trade volumes or by converting trade fee earnings while developers are allocated DEED based on DEX development milestones on a monthly basis. 

DEED has a fixed total supply of 1.24 billion tokens that can be earned by trading on DEX or by contributing towards development of the exchange. 1 billion (80.65% of total supply) tokens are reserved for users trading on DEX while 0.24 billion tokens (19.35% of total supply) are reserved for developers to be allocated over a period of time. Users earn 1 DEED for every $1 traded on DEX, which implies that 1 billion DEEDs would have been earned & distributed when Deed DEX attains $1 billion of trading volume. Developers are allocated DEED on a monthly  basis to compensate for the time & effort contributed towards DEX development at the rate of 0.6% of total supply available for users (6M tokens monthly) over a period of 40 months which is expected time for DEX to hit $1 billion in trading volume & completion of full decentralization phase. 


## Deed DAO

All users of Deed DEX holding DEED shall be default members of Deed DAO with 1 voting per DEED held as on date. Deed DAO would be the primary governance body consisting of DEX users & developers with powers to vote on any changes in DEX contract as defined in contract upon creation or launching new contracts to interact with DEX like staking or yield farming. It is important to note that members of DAO need to be users of DEX with at least $1000 in trading volume over the last 6 months from date of proposal while holding DEED tokens that defines their voting power. The limits defining users for DAO can be revised by voting proposals. Any DAO member can create a proposal which when ratified by at least 10% of total voting power, would open for voting to all members for 14 days. Any proposal ratified by 51% or more of voting power in circulation at the time of the vote shall have passed, while a proposal ratified by less than 10% voting power would be annulled. Some functions of Deed DEX that DAO members can vote on-

* Launching Deed DEX contracts on new blockchains
* Approving trusted third party oracle integrations with Deed DEX
* Modification in bidding mechanism
* Setting trade fee range based on asset types
* Determining DEX earnings distribution ratios between asset owners & traders
* Updating DEED token rewards computations ($1 trading volume= 1 DEED)
* Adding or removing an asset type on exchange (gold, fiat currencies etc)
* Implementation of staking & yield farming contracts
* Listing DEED on other CEXs or DEXs


## Links

1. [Pre-launch](https://deed.so/pre-launch)
2. [Prototype](https://deed.so/version-test)


