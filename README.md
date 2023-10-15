# deed-PWA
Deed is a PWA app where anyone can tokenise an internet/online asset &amp; earn passive income by launching trade pools.

## Introduction

What is the value of Internet? ---Is it the total market cap of Google, Meta, Amazon & Apple's of the world or is there a way individual assets hosted on the internet can be valued independently from the Big Tech hosting such assets. Millions of entreprenuers & creators who create & own assets on the internet be it a youtube channel or a shopify store or a game app on play store, <b> don't have any way of discovering true market values of their assets </b> even though many such assets are generating huge cash flows. <b> Deed aims to decouple value of internet assets from Big Tech platforms & offer a way for asset owners to not only determine market values of their assets, but also earn passive income from tokeinsed version of their assets on the side. </b> Discovery of true market values of internet assets, can open up gateways to a new decentralised financial system for these assets.


## Design

### Problems


1. Tokenization is complex with multiple barriers around paying gas fees & authorising individual transactions.
2. Launching liquidity pools (trade pools) is neither easy nor intitutive even for many crypto natives.
3. Tokenisation & trading of tokens happens on separate apps, causing pain to switch.
4. Tokensation & liquidiy pools are disconnected from real world assets.
5. Mobile based tokenisation & trading has better UX, but crypto apps mostly get rejected on app stores.
6. Rug & pull scams are common on popular AMMs with no safeguards for traders.

### Goal

To build Deed as a Progressive web app that makes the experience of tokenization & trading of tokens connected to internet assets easy, intuitive & safe.

### Prototype (screens)




https://github.com/robbin2102/deed-PWA/assets/103725283/5ce3259c-4224-425d-87ee-4287b071ea3e





See [Prototype](https://deed.so/version-test) for all screens.

## Trade Pools

Trade pools are smart contracts that hold an equal amount of tokens (connected to an internet asset) & another cryptocurrency native to chain for facilitating exchange & trade utilities. [Solana token swap program](https://spl.solana.com/token-swap) has been used as base contract with some customizations around liquidity lock mechanisms & bonding curve to create token swap pools of assets tokenised on Solana. A key advantage of using Solana is that the cost of deploying token contracts & launching trade swap pools is almost 100x lower than other L1 & L2s. This enables Deed App to provide wallets that can have unlimited transactions at ZERO gas fee where the gas cost of any transaction on Deed app is borne by a central wallet of the app & not the user thus making transactions gas free for the user improving UX. Users can seamlessly trade through embedded wallets with 0 gas fees.

### Trade fee

Asset owners can set custom trade fees for every trade pool in the range of 1%-10%, based on projected trading volumes & estimated earnings they would like to attain. Custom trade fees opens up use cases for multiple type of assets where owners might want to keep a low range expecting high volumes & vice-versa. The range between 1-10 is experimental & subject to updates based on user feedbacks.

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


### Burners

To avoid asset owners from minting more supply outside Deed, the freeze authority of a token contract connected to an internet asset is alloted to a swappool account that gets triggered by webhooks watching for new mints to freeze the supply created. The objective again is to reduce probability of rug pulls, though inherently now that internet assets are connected to tokens, reputation of owners act as natural detterents against rug pull schemes. Another approach now negated was to put a cap to total supply, but since utilities & use case of tokens connected to internet assets would evolve, its better to have an open ended system with certain safeguards in place.

### Bonding curve

Deed uses a custom bonding curve function instead of constant product. 

```bash

// Function to calculate the price of a token based on the custom bonding curve

function calculateTokenPrice(tokenSupply) {
    return initialPrice + (priceIncrement * Math.pow(tokenSupply, exponent))
```


## Tokenization

Users can deploy gasless token contracts & mint initial supply to be locked in trading pools. Tokenization in version 1.0 (MVP) is a plain vanilla contract with freeze auth delegated to swappool burner accounts. Additionally public beta versions (2.0 & later) would consist of Asset validation using zk-proofs. 


### Asset validation 
It is important to uniquely attribute an online asset to its on-chain token in a way that users can validate to gain trust about assets represented by those tokens. On blockchains anybody can deploy token contracts with excatly similar symbols & names thus creating confusion for users to identify tokens that are connected to real assets. So asset validation is an important feature not only for users of Deed, but overall for the crypto ecosystem dealing with tokens represented by real assets. The implemnatation of this feature has ben reserved for later versions. 


## Asset data

Traders need real time data about an asset's fundamentals- users, visitors, customers, transactions,  subsribers, followers, views, impressions, earnings etc, to make efficient trading decisions on the price of assets. Deed app pulls data from platforms hosting assets with user authentication to link & show case the data to traders within the app. This is important to determine fair market values of assers & further mechanics on delivery of data can be controlled by owners with a score on transparency. For example- Some owners may choose to declare data real time (a score of 10/10) while others may choose not to ( a score of 2/10). Transparency scores is a mechanism help establish trust between asset owners & traders.

## Links

1. [Pre-launch](https://deed.so/pre-launch)
2. [Prototype](https://deed.so/version-test)


