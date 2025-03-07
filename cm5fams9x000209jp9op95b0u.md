---
title: "How to Build Capital-free Approve-less Flash Arbitrage Bots Across All Networks"
datePublished: Thu Jan 02 2025 12:19:33 GMT+0000 (Coordinated Universal Time)
cuid: cm5fams9x000209jp9op95b0u
slug: how-to-build-capital-free-approve-less-flash-arbitrage-bots-across-all-networks

---

## Introduction

Hi Degens,

This article series is specifically designed to explain the concepts and mechanisms of Arbitrage Bots in the DeFi field. Arbitrage bots earn a lot during market fluctuations, but the question is, how to build one?

> I assume you have basic knowledge of DeFi, Liquidity Pools, AMMs, and DEXes.

## What is Arbitrage?

Arbitrage is a way to balance the price discrepancy of an asset to its global market price. In a real-world example, it is like buying a brand new car for $8000 since its dealer price is $9000. Even now, you profited $1000 by just buying this car for $1000 lower than its dealer price, but where and how is this opportunity happening, and how to catch and take those in DeFi?

Let me explain. In DeFi, we are selling our tokens like USDC and WETH for another token/tokens. In return, we expect to get another type of asset like in the real world. For example, exchanging 50000 USD for a BMW M3, like here, we exchange X amount of Token for Y amount of another Token.

In depth, for example, we exchange 50000 USDC for 10 WETH, we send 50000 USDC tokens from our wallet to an X contract address, and this contract address or another address is sending us 10 WETH that we were expecting, and in this example, we have bought WETH tokens for $5000 per unit of WETH. If the market price of WETH were $5100, we would have profited $1000 worth of WETH tokens.

Okay, that was the basics of arbitrage in DeFi, but the most important thing to notice in those examples is **we must have 50000 USDC to buy 10 WETH** but what if we haven’t?

That is the place where flash loan, flash swap, collateral swap, and debt swap techniques come into play.

Let’s learn what those are.

## What is Flash Loan / Flash Swap / Collateral Swap / Debt Swap?

* **Flash Loan**: This is a type of loan that doesn't require collateral and must be borrowed and repaid within the same block. It allows users to access large amounts of capital for arbitrage or other purposes without needing upfront collateral. If the loan isn't repaid within the same block, the entire transaction fails, and everything is reverted to its original state.
    
* **Flash Swap**: This is a feature of token exchange transactions, commonly used in AMMs and DEXs. It lets the receiver get the output token first and send the input token later, all within the same block. This feature is available only to *Contracts*, allowing them to turn a regular token exchange into a flash swap transaction. Unlike a flash loan, if we want to borrow some USDC, we need to swap from the pair's other token to USDC. To enable a flash swap transaction, we simply send data longer than 0 in our function call to the liquidity pool contract. This way, the liquidity pool contract will trigger our **pre-written callback** **function** on our contract, allowing us to perform additional processes within the same block with our output USDC.
    
* **Collateral Swap**: This method involves using a lending application to make a profitable transaction by changing the collateralized asset into another one. More details will be explained in upcoming articles.
    
* **Debt Swap**: This method involves using a lending application to make a profitable transaction by changing the borrowed asset into another one. More details will be explained in upcoming articles.
    

With those techniques, we can easily take profits from liquidity pools by not depositing or using any money as capital.

## Implementation of Flash Techniques

To use Flash Loan / Flash Swap and all the other flash techniques, you first learn where to apply those techniques, let’s learn liquidity pools.

## Liquidity Pools

Liquidity Pools are the exact places for our arbitrage bot to earn money during market fluctuations. You have to know the formulas, types, callbacks, fee rates, and almost all the things related to liquidity pools to become a master of them. Somehow, calculation and formulas are easy for some kind of liquidity pools but some of them require really high effort to calculate input and output exactly. So, let’s dive in.

> Liquidity Pool = LP (Abbreviation of Liquidity Pool, Convention in DeFi)
> 
> DEX = Decentralized Exchange
> 
> AMM = Automated Market Maker
> 
> CPMM = Constant Product Market Maker
> 
> CLMM = Concentrated Liquidity Market Maker
> 
> sqrtPriceX96 = Square Root of The Price times 2 to the power of 96

1. **Uniswap V2 Pools (CPMM) (Fee Rate 0.30%)**: aka Constant Product Liquidity Pools.
    
    Uniswap V2 is the starter kit of an arbitrageur to learn the dynamics of a pool.  
    Pools have Constant Product Liquidity Formula =&gt; X *Y = k  
    X = reserve of token0, Y = reserve of token1, k = Constant.  
    Which makes arbitrageurs calculate the exact output amount* for a given *exact input amount*.  
    Uniswap V2 Pools have a fixed Fee Rate of 0.30%, during every swap() function execution, 0.30% of your input token will be taken as a pool fee to disperse it to liquidity providers. For example, 0.30 WETH will be taken as a pool fee when your input is 100 WETH, so it is easy to calculate.
    
2. **Fork Uniswap V2 Pools (CPMM) (Different Fee Rates)**: There are some other Dexes or DApps that have different fee tiers.  
    PancakeSwap V2 has a 0.25% pool fee on their V2 pools.  
    Antfarm Finance V2 has 1%, 10%, 22.2%, 25%, 50%, 100% fee tiers on its V2 pools, goes like that.  
    Those DEXes have the same infrastructure to make swaps on a pool, so the only difference here is the fee tiers, but sometimes, like in the Antfarm Finance V2, some functions may have not been implemented completely from Uniswap V2.  
    Thus, in order to take profit on those pools, you must check the input parameters, function content, callbacks, and return values cautiously to avoid losing your money for 1 wei :).
    
3. **Fork Uniswap V2 Pools (CPMM) (Dynamic Fee Rates)**: Unlike the fixed fee rate on V2 pools, there are some DEXes having dynamic fee rates and continuously changing the fee rates for both token0 and token1. For example, Dyson Finance V2 Pools have a dynamic fee range feature. Thus, fee rates for both token0 and token1 are being updated every 12 seconds, so you must fetch the current fee rates to calculate input and output which makes your app make a lot of requests.
    
    * Normally, if you aren't a wallet (EOA: Externally Owned Account) and the address calling the swap() function is a contract, then you can exchange tokens without any approve permission to give any pool. But, in Dyson Finance V2 pools, we have to give approve permission directly to the ***pool***. So, that's crazy, but they built it like this, anyway let's continue.
        
4. **Uniswap V3 Pools (CLMM) (Fixed Fee Tiers)**: aka Concentrated Liquidity Pools. The second station of an arbitrageur is Uniswap V3.  
    LP formula of Uniswap V3 Pools =&gt; (X + L/sqrt(Pb)) *(Y + L* sqrt(Pa)) = L^2  
    X = Real Reserve of token0, Y = Real Reserve of token1, L = Liquidity.  
    Pa = Lower price boundary, Pb = Upper price boundary.  
    Apparently, that is a bit more complicated than Uniswap V2 Pool. Therefore, the calculation process of the exact output amount for a given exact input amount is a bit hard but not impossible.  
    I already invented a formula that calculates those things without any call to be made to the pool contract in 2 ms. So, that's crazy but it is not impossible to calculate the input and output without making any request.  
    Uniswap V3 Pools have 4 fee tiers =&gt; 0.01%, 0.05%, 0.30%, 1%.
    
5. **Algebra Pools (CLMM) (No Fee Tiers)**: Built with the same infrastructure of Concentrated Liquidity Pools but have some different fee mechanisms.  
    There are 2 fee types for this kind of pool; Base fee, community fee.  
    Base fee, as far as my knowledge, 0.01%  
    Community fee, as far as my knowledge, it can go up to 0.25%  
    Except for those fee rates, and the input differences while calling the swap() function on the pool, there is no exception left for us.
    

There are tens of different types of liquidity pools I'm 100% sure about that, but since most of them are not worthy to inspect their codebase and now enough volume isn’t made on those liquidity pools, I don’t choose to take profits from them.

---

## Arbitrage Bot Types

1. **Capitalized Arbitrage Bot**: This bot needs some tokens to exchange across pools.  
    It is the simplest arbitrage bot, you just give some token for another token.  
    — Benefits =&gt; You can set your minimum profit limit as low as you want because holding directly the token balance on your contract allows you to be flexible  
    — Losses =&gt; In the market fluctuations, you can lose/earn money and it can occur your capital goes down, so at the end of the year, maybe you may have lost all the profit while market fluctuations.
    
2. **Triangular Arbitrage Bot**: This bot uses 3 pools to profit by using different tokens and the important thing is input and/or output tokens must be connected with the next pool.
    
3. **Cross-Chain Arbitrage Bot**: I love the idea behind this arbitrage bot, but it is difficult to build because expecting the exact amount after Cross-Chain Arbitrage, condition might not be completely met and sometimes you can lose your money a bit. But, in my opinion, this can be the most profitable arbitrage bot, even you can find little arbitrages across chains using the bridges, bridge aggregators, and let’s just think if the bot does this automatically and finds the arbitrages across all the chains.
    
4. **Liquidation Arbitrage Bot**: This bot is used on Lending platforms such as AAVE.  
    The idea behind this bot is when your collateral becomes under-collateralized, this bot participates to liquidate your loan and gets the profit from buying collateral as discounted. I haven’t used or experienced that bot before, but this is a hell of a good arbitrage bot, you should research how it works.  
    [https://blog.amberdata.io/performing-liquidations-on-the-aave-defi-lending-protocol](https://blog.amberdata.io/performing-liquidations-on-the-aave-defi-lending-protocol)
    
5. **MEV Arbitrage Bot**: These bots aim to capture value by front-running or back-running transactions within a block. They look for arbitrage opportunities by manipulating the order of transactions in the mempool.
    
6. **Flash Loan Arbitrage Bot**: Flash Loan bots borrow huge amounts of money from Lending DApps, LPs, others, then use it to take profits on LPs, and other places.  
    Flash Loan makes you borrow X amount of token and repay X + X \* flashLoanFee within the same block, with that opportunity, you have nearly unlimited capital to take profits, but the thing is you have to return the same asset to the contract that you have borrowed from, flash loan can only work with the same asset, so you can’t repay with another token.
    
    — Benefits =&gt; at AAVE, there is a flash loan fee, but at Balancer Vault, there is no flash loan fee, so you can borrow 15 Million USDC, then repay it as 15 Million USDC, that’s it. Also, in some cases maybe returning the same asset to the address you have borrowed from is beneficial for your strategy and can make you more profit than other bot types.
    
    — Losses =&gt; as I said, AAVE takes a flash loan fee, so you have to pay a fee as a percentage for your loan amount, and that can be disappointing while borrowing a huge amount of tokens from AAVE or any other platforms.  
    The second important thing is you have to return the same asset that you have borrowed. If you are doing arbitrage on LPs, probably you have to make another exchange to swap your profited asset to your borrowed asset, and it will cause price discrepancies and not finding any pool to convert your borrowed asset for that much amount of token.
    
7. **Flash Swap Arbitrage Bot**: Well, that is my favorite bot type and I use only that method currently in my arbitrage bot project. Okay, let me explain to you.
    
    — When you make a normal swap with your EOA wallet in Uniswap V2 / V3, you have to send your WETH first and wait for the USDC transferred to you.
    
    — When you call the swap() function on the pool contract with your Contract in Uniswap V2 / V3 pools, unlike the normal swap with EOA, first we receive USDC, and then send WETH.  
    The pool contract is designed to transfer USDC to your Contract optimistically\*\* and then it is expecting the exact amount of WETH.
    
    So, that gives us opportunities =&gt;
    
    Let’s say there is a $500 arbitrage on Uniswap V2 Pool, 1 WETH to 3000 USDC
    
    We are calling swap() on Uniswap V3 WETH/USDC Pool, 2500 USDC to 1 WETH  
    At the beginning of the transaction, the V3 Pool directly sends us 1 WETH  
    After that, we are calling V2 Pool’s swap() function and sending 1 WETH  
    Taking 3000 USDC from V2 Pool  
    Sending 2500 USDC to V3 Pool  
    Gotcha! We have earned 500 USDC!
    
    Thus, we are able to make a profit without no capital, but the hardest part here is you have to prepare everything from end to beginning because the process starts from the V3 pool and ends with the V2 pool.
    
8. **Debt Swap Arbitrage Bot**: This bot works on Lending DApps and instead of selling profited tokens to your base token to pay back the loan, you are changing your debt token to profited token on Lending DApp. It is a bit hard to calculate how much is going to pay for X token, but once you figure out the whole formulas, everything is going to be okay. This way of doing arbitrage is almost 50% profitable compared to the regular Flash Swap Arbitrage strategy.
    
9. **Collateral Swap Arbitrage Bot**: The same methodology works on this bot as is on Debt Swap Arbitrage Bot, but the important thing is collateral\*\* must be changed in this strategy. Thus, you’ll have to figure out LTV (loan-to-value rate) calculations, how many collateral tokens must be swapped to X token to make arbitrage profitable, collateral-to-debt and debt-to-collateral relations and calculation formulas, after everything is done, you are good to go for taking profits.
    

---

## Arbitrage Bot Infrastructure

1. **Concurrently Execution**: Hard topic to explain, my bot can work on 148 chains, 800 Dexes, and probably more than 30000+ pools. But, if I try to run my bot with 800 Servers on my PC, probably my PC will be gone. Even if you try to run that amount of servers on your local PC, it is possible to break your PC by just thinking. So, we need Concurrently Execution.
    
    To explain simply, every dex’s code file must consume RAM as low as it can, and to do that, every request made in this file must be lowered and must be quick, and after the request period is over, the dex’s code file must change its position to listening, instead of requesting.
    
    In the listening mode, we are only listening to the pools and waiting for the real opportunity for profits, the code will never execute or request process unless our listening mechanism notices us there is a profit to take from the pool.  
    Thus, we are keeping all of our dexes’ code files in the stand-by mode like in our TVs, and it doesn’t let any unnecessary execution while market fluctuations, not every time, but it generally doesn’t let :)
    
    In the requesting mode, we are making all the input, output, parameters, data, and other variables’ calculation by making requests as low as we can, currently, I am making 1 mistake only for transaction simulation.  
    After requests are done, we are executing the transaction and sending it to the blockchain
    
    — Concurrently Execution was the hardest thing to build for me because you have to catch then rule every exception properly for system reliability, I hope you can build one with those concepts of working schema too.
    
2. **Mempool Listening / Pool Listening**: The second most important thing and also required for our listening mode is listening to the pool contracts’ events.  
    In Uniswap V2 and Uniswap V3, whenever the swap() function is called and the transaction succeeds, the **swap** event is fired.
    
    Uniswap V2 =&gt;
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726338281300/6d1554b0-bec8-4b47-ba34-30fa428d4682.png align="center")
    
    Uniswap V3 =&gt;
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726338322779/1a634c3e-0f91-4d33-9299-09478aa19ce8.png align="center")
    
    The thing is, we have to catch those event values directly at the time when the pool fired. It can be achieved using the addEventListener method in ethers v5.7.2.  
    I don’t use ethers v6 because I didn’t like the mechanism of listening, ethers v5 is way better than v6 according to pool listening.
    
    — Mempool Listening  
    You can directly listen to every transaction sent to the blockchain at the time of being sent, that means the transaction isn’t approved yet and it is still waiting to be in a block, that gives us an opportunity to make sandwich attacks, but we of course do not prefer that.
    
    Instead of this, you can decode\*\* transaction calldata with your special decoder code and you can see what is going to happen in which pools and how much gap will be appeared after the swap, with this way, you can prepare your transaction for the profit that will be occurred after the approval of this pending transaction and send it right after this pending transaction is approved.
    
3. **Quoter Contract Versions & staticCall for Input/Output**: Uniswap V2 Based Dexes do not use Quoter contracts, because you can get exact input or exact input by asking the Router Contract with getAmountIn() and getAmountOut() function.
    
    But, it is different in Uniswap V3 Based Dexes, calculating the exact input and exact output isn’t easy and Uniswap fixed this problem by directly calling the pool’s swap() function **statically** and catching swap() function’s return values, and finally returning to you for your specific request.  
    I think this is the easiest way to get input/output for a situation, I think the same with Uniswap devs for that topic :)  
    But, with that way, you have to wait until this Promise request resolves and probably will slow down your code.
    
    Therefore, if you want to be a really degen arbitrageur, you have to calculate the input/outputs only using mathematical functions, you shouldn’t wait for any request, and make sure you have done everything and wrapped data and already sent to the blockchain before everyone else, because there are tons of arbitrage bots over there.
    
    — Quoter Contract Versions: As far as I know, there are currently 4 different Quoter Contract Versions, which means 4 different types of parameter types for each function on the Quoter Contract.
    
    — Quoter V1 =&gt; [https://etherscan.io/address/0xb27308f9F90D607463bb33eA1BeBb41C27CE5AB6#writeContract](https://etherscan.io/address/0xb27308f9F90D607463bb33eA1BeBb41C27CE5AB6#writeContract)
    
    — Quoter V2 =&gt; [https://etherscan.io/address/0x61fFE014bA17989E743c5F6cB21bF9697530B21e#writeContract](https://etherscan.io/address/0x61fFE014bA17989E743c5F6cB21bF9697530B21e#writeContract)
    
    — Quoter V3 =&gt;  
    [https://zkevm.polygonscan.com/address/0x55BeE1bD3Eb9986f6d2d963278de09eE92a3eF1D#writeContract](https://zkevm.polygonscan.com/address/0x55BeE1bD3Eb9986f6d2d963278de09eE92a3eF1D#writeContract)
    
    — Quoter V4 =&gt;  
    [https://lineascan.build/address/0xcE829655b864E56fc34B783874cf9590053A0640#writeContract](https://lineascan.build/address/0xcE829655b864E56fc34B783874cf9590053A0640#writeContract)
    
    You can see the difference in parameter types.
    
    — Static Call  
    QuoterContractV2.quoteExactOutputSingle.staticCallResult({  
    tokenIn: tokenOut\_address,  
    tokenOut: tokenIn\_address,  
    amount: amountIn,  
    fee: pool.pool\_fee,  
    sqrtPriceLimitX96: sqrtPriceLimitX96  
    });
    
    Using ethers v6.7.2, you can make a static call request to the quoter contracts in this way.
    
4. **Callback Functions**: Each Dex has its own unique callback function name, for example, in PancakeSwapV2, when you call the swap() function in any of the pool, the pool contract will call =&gt;
    
    \=&gt; pancakeCall(address sender, uint amount0, uint amount1, bytes calldata data)
    
    Because of that reason, PancakeSwapV2 expects you to inherit its [IPancakeCallee Interface](https://github.com/pancakeswap/pancake-swap-core/blob/master/contracts/interfaces/IPancakeCallee.sol) to complete the transaction on your contract successfully.
    
    In the meantime, some dexes like SushiSwapV2, can use the same callback function name to continue the transaction on your contract
    
    \=&gt; uniswapV2Call(address sender, uint amount0, uint amount1, bytes calldata data)
    
5. **Batch Transaction**: Combining multiple transactions and contract calls into a single transaction. We need Batch Transaction because our transaction must complete every call within the same block. So, we can’t take the risk of making 4/5 calls one by one within the same block, because it’s not a guaranteed way to do and probably will revert. Because of that, every call must be lined up and at the end of the day, we will send that transaction to the blockchain to execute everything within the same block successfully.
    
6. **Data Encoding**: Let’s say we found a $500 profit on Uniswap V2 with 1 WETH input and 3000 USDC output, assuming 1 WETH price = 2500 USDC, and to make that with no capital, we have to use flash loan / flash swap / debt swap / collateral swap strategies, let’s use flash swap with Uniswap V3.  
    Therefore, we have to calculate how much USDC will be required to get the exact amount of 1 WETH from that Uniswap V3, let’s decide it 2550 USDC.
    
    From now on;
    
    * Uniswap V2 Pool’s swap() data =&gt; Function Selector + Parameters Encoded
        
    * Uniswap V3 Pool’s swap() data =&gt; Function Selector + Parameters Encoded
        
    * Callback =&gt; Used to determine which tokens will be sent to pool/pools
        
    * flashSwapData =&gt; Used to call every Uniswap V2 & Uniswap V3 pool without no interface, just uses pure call() function, which means I built it to work interoperable across every dex. Your contract’s function selector + parameters (Unified data of those data)
        
    
    To continue our sentence, we have to specifically determine every input, output, sqrtPriceLimits, minimum, maximum token inputs/outputs, function selectors, universally executable data callbacks. Also, for each type of dex, you have to make a standard and you have to be hitched to those standards while preparing data and expecting to work those data piles work on every dexes
    
7. **Aggregation**: That is the thing that I am currently building and it’s the craziest thing for me so far, it’s not that hard but simply you are gathering all the profits from let’s say 25 pools and doing all the things in one transaction, so all the callbacks, parameters, and no revert statements must be met in that situation, So, it is difficult to build, but is a lifesaver to profit from low-profit providing pools.
    
    Actually, there are aggregator dapps like Symbiosis Finance, 1inch, OpenOcean, ParaSwap, etc. They are doing this thing but doing this probably consumes a lot of resources from your provider. Therefore, it is not easy to build for me as fast and effective.
    
8. **Approve-less Work**: Thanks to Uniswap V2 and Uniswap V3 swap() function design, we can exchange tokens between our contract and pool without giving **no approval**
    
    It is not available on every dexes but it is available almost 90% of the Uniswap V2 and V3 forks. Some dexes still need approval for token exchanges even we are calling directly pool’s function, that’s crazy.
    
9. **Non-Hackable Contract**: Technically, it’s not possible to hack my contract thanks to the flash swap technique and my minimum output return is controlled within my contract.
    

---

## Libraries / Modules / Setups

1. **Fetcher V2**: It basically gets all the pools that arbitrage can be made in those pools for all types of V2 pools from the factory contract of projects.
    
2. **Fetcher V3**: It basically gets all the pools that arbitrage can be made in those pools for all types of V3 pools from the factory contract of projects.
    
3. **ABI Handler**: There are a lot of types of pools so we have the compound and make an efficient JSON file to handle every pool’s functions to call.
    
4. **Decoder**: It decodes the data which is generally the failed transactions’ data to inspect what has happened and what calculated wrong in the process. It decodes long hexadecimal text into a human-readable form for inspection.
    
5. **Contract Functions**: It is used to EMERGENCY WITHDRAW, sending tokens to the wallet, making any transaction on the contract, and using a function on the contract.
    
6. **Oracle**: It uses Pyth and/or Chainlink and/or any other oracle to get the market prices of the coins that I selected. So, currently, I use Pyth live data, and it works perfectly.
    
7. **Market Prices**: It gets the market price of the coins from CoinMarketCap and CoinGecko, I use it but it doesn’t have data freshness but anyways, I use it to get the price of tokens that aren’t in the oracles.
    
8. **Library**: All the functions to get exact output, calculate optimal input, and all the things that can be called in my project are stated in this folder.
    
9. **Fee Data**: Basically it gets the fee data (gwei) for each chain.
    
10. **Common***:* It holds all the projects’ data and everything about a chain and its tx type, module type, mempool availability, and more.
    
11. **Creator**: It creates the .js files directly from scratch and makes all the content from scratch for a given chain, and then, it prepares everything for every project in order to rules in the Common/Chain.js file. Basically, if there is 500 DEX in the Ethereum chain, you can create all of their arbitrage files in just a few seconds with that script.
    
12. **GeckoTerminal & DexScreener & DexTools**: I use them to get the factory addresses, projects’ names, pool types, all the chains’ data, chain’s ID, chain names, and decision the type of the pools are made by my code that calls some function on those pools to decide whether it is V2 or V3 or any other kind of pool.
    
13. **Provider Changer**: If you will be arbitraging across a lot of chains, you will need to have a lot of backup nodes/providers to successfully send the transaction to the blockchain, this script works to handle the request per limit, and conflicts of the request, current request made by the pools and handling the traffic, if the provider/node exhausted, chain with the new one and resetting the chains’ servers.
    

## Conclusion

This concludes our introduction to building capital-free flash arbitrage bots across multiple networks. While we've covered the basics, this is just the start of an exciting journey into the world of DeFi. In upcoming articles, we’ll explore advanced techniques, optimization strategies, and implementation tips to help you improve your bot for real-world performance.

Here’s something to consider before we continue: **How can you take profits from 50 liquidity pools at the same time within one block when some of them don't match your token pairs? That's where Aggregation comes into play.**