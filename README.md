## Repo Contents
This repo containes a DEFI arbitrage bot which implements flashloans to maximize arbitrage profitability between the uniswapV2 exchange along with some of its forks such as sushiswap, crowswap, sakeswap and shibaswap. This read me contains some theoretical information around aribtrage, flashbots and some of the mathematics involved aswell as detailed instructions on how to clone this repo and set up the installation in order to run the bot yourself with no coding required.

1) A bot written in JS that observe the prices changes on a pre user-defined liquidity pools at Uniswap V2 / Sushiswap crowswap, sakeswap and shibaswap to determine if its possible to make a profit buying tokens cheaper at one exchange to selling them after for a bigger amount in the other, keeping the difference (profit). Uniswap, like all the others exchanges, charges on you a fee for using their services (swapping tokens in this case), this fee currently is 0.3%. To see more on how flashswap or normal swaps works visit the [Uniswap docs](https://uniswap.org/docs/v2/).

2) A demo that you can easily run to see the bots in action. Basically consist of forking ethereum mainnet locally, run the demo script that do the set ups for you and execute the bots.

3) Theory of Arbitrage and some proposed improvements for possible production stage of this bort.

First it will be explained how to install the required tools (probably you have already installed some of them, feel free to jump to the sections that you wish). Then I introduce how to run the demo, here assume that you are kind of newbie in blockchain and you don’t understand quiet well whats happening so its deeply explained. After, a very basic guide line to put the bots to work on an ethereum network, mainnet or testnet, I assume you know what you are doing at this point. Finally some improvements for a possible production stage and useful resourses are given.

# Instillation
to use this code follow the detailed guide below. I have went the extra mile here to really explain eveeything so that anyone can run this code error free regardless of experience.

### Software requirements
Before you can actually run the code there is two pieces of sogtware that you are required to install on your mchine. Those are

1) Node.js
2) Truffle suite

The first is Node.js. Node js is a javascript backend runtime enviornemt and we need it to run our arbitrage bot script. The second is the truffle suite. This pretty much is a bundle of packages for blockhain development. It comes with solidity, the truffle testing enviornemtn and a python for running simple web servers. To install Node.js go to https://nodejs.org/en/ and install the `Current Build`. A screenshot of the correct site is shown below

![Alt text](../../../Documents/tempsnip.png?raw=true "Title")

<img src="https://drive.google.com/drive/u/1/my-drive/tempsnip.png"/>


# Automated Market Maker Arbitrage
**The current DEX ecosystem is mainly under one AMM family called constant function market makers** 
As a market maker, providing liquidity is a tedious task, it often involves locking up significant capital, programming a trading bot and building up a feasible strategy. It is often in the hands of institutional providers where they have the capability and incentives to build such initiatives. 

The AMM structure creates possible ways for any individual to become passive market makers without worrying about the technical details for market makings.

There are a few different strategies in creating that AMM structure, we call it the constant function market makers (CFMMs), under the CFMMs, there are a few different substructures with their unique characteristics.

1. Constant product market makers (CPMMs) - Uniswap
      
      X * Y = K

2. Constant mean market makers (Generalised CPMMs) - Banlancer Protocol

<img src="https://cdn.substack.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F8b2f3800-d13b-4cbf-80d9-e01e1edc51ee_254x105.png" />

3. Constant sum market makers (CSMMs) - Unfit for the decenralised exchange use case

<img src="https://cdn.substack.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F8b2f3800-d13b-4cbf-80d9-e01e1edc51ee_254x105.png" />

4. Hybrid CFMMs - Curve finance, Bancor V2, MCDex

- Curve Finance Amm 
<img src="https://cdn.substack.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F8b2f3800-d13b-4cbf-80d9-e01e1edc51ee_254x105.png" />

- MCDex amm
- P = i + Beta * i * E

Total return = trading fees + impermanent loss/gain.

Locking up capital and thus by providing liquidity to the pool, the liquidity providers (LPs) need some incentive to do so. Under the current incentivised structure, the LPs will get reimbursed for their capital lock-up via trading fees generated by the traders and arbitrageurs. However, with the design of AMM structure, the LPs can generate a gain/loss called impermanent gain/loss which at the most of time would be a loss and significantly reduce their overall returns.

The initial AMM structure does generate sufficient liquidity in the DEX ecosystem, however, due to the design and actual implementation, LPs sometimes would generate negative overall return due to impermeant loss.

For example, if LPs started to provide liquidity on DAI in the beginning of the year, LPs made +10.42% return to date netting impermanent loss. However, if instead LPs chose HEX trading pairs to provide liquidity, the LPs would suffer a net loss of -55.14%. This shows how impermanent loss can greatly affect the LPs’ overall return.

There are a few proposals to reduce or theoretically eliminate the impermanent losses and, in the meantime, further reduce slippages. For example, Curve is using hybrid CFMMs targeting stablecoins or wrapped assets to provide significantly better liquidity for stablecoins. Bancor V2 and MCDex also proposed their own solutions to counteract the issues with impermanent loss.








## The rationale
Suppose that we want to do arbitrage transactions on a token pair in the form of TokenX/WETH. The TokenX/WETH pair first of all needs to exist on at least two exchanges in order for us to be able to arbitrage. In this case WETH is known as the base token. Note that the base token can be any token some common base tokens are USDT/USDC/DAI etc. Howeevr this bot impleemnts DAI/WETH with WETH being the base token

TokenX in this case is called the quote token. After we preform the arbitrage the quote tokens wont be reserved.Instead the base token is reserved this means that our profit is denominated in the base token. In the case where a token pair consists of two base tokens either can be reserved. See the TradeOrder.sol smart contract logic for this.

The profitability of arbitrage can be maximized by using a `flashloan` on the uniswap exchange or sushiswap exchaneg for that matter.Lets suppose that pair0 and pair1 are two pairs of the same tokens on different exchanges (DEXES). Once the price diverges between the exchanges we can in theory, preform optimistic arbitrage. We can call index.js to start the arbitrage. The contract script uses the uniswapv2 contracts, namely the V2Factory and V2Router to calculate the price of the pairs denominated in the quote token on the different exchanges. If the script successfully finds a price mismatch then we carry on to calculate an estimation of the gas fees required for doing a flaswap. here we also take into consideration the 0.03% fee which we must pay to uniswap. If anfter all of these calculations a profitable trade still remaisn then we call the SlashBot.sol contract. This contract allows us to borrow some quote tokens from pair0. We can let the borowed amount be x. The contract will neeed to repay the debt to pair0. The debt can be denominated in the baseToken. This is actually a very useful functionality of uniswap which saves us having to develop the logic ourselves. When we take the loan we sell all of the borrowed tokens on pair1. The contract gets base tokens ofamount y2. We then immediately pay back the uniswap contract to pay off the loan and our flashswap contract gets to keep an amount of y2-y1.

## Closer look at the Maths
Before i explain the instillation lets look at some of the fiancial maths. We can actually write an algorithm that can calculate the amount of x so we can get as much profit as possible. Consider the formulae below

Consider the initial state of pair0 and pair1
|                     | Pair0 | Pair1 |
| :-------------------| :-----| :-----|
| Base Token Amount   |  a1   |  b1   |
| Quote Token Amount  |  a2   |  b2   |

In this case we can caluate the amoutn to borrow to maximise our proft as:

<img src="https://latex.codecogs.com/svg.image?\Delta&space;a_1&space;=&space;\frac{\Delta&space;b_1&space;\cdot&space;a_1}{b_1&space;-&space;\Delta&space;b_1}&space;\&space;\&space;\&space;\Delta&space;a_2&space;=&space;\frac{\Delta&space;b_2&space;\cdot&space;a_2}{b_2&space;&plus;&space;\Delta&space;b_2}" title="\Delta a_1 = \frac{\Delta b_1 \cdot a_1}{b_1 - \Delta b_1} \ \ \ \Delta a_2 = \frac{\Delta b_2 \cdot a_2}{b_2 + \Delta b_2}" />

The amount borrowed Quote Token are some, so `Delta b1` = `Delta b2`, let `x = \Delta b`, then the profit as a function of x is:

<img src="https://latex.codecogs.com/svg.image?f(x)&space;=&space;\Delta&space;a_2&space;-&space;\Delta&space;a_1&space;=&space;\frac&space;{a_2&space;\cdot&space;x}{b_2&plus;x}&space;-&space;\frac&space;{a_1&space;\cdot&space;x}{b_1-x}" title="f(x) = \Delta a_2 - \Delta a_1 = \frac {a_2 \cdot x}{b_2+x} - \frac {a_1 \cdot x}{b_1-x}" />

We wanna calculate the x value when the function get a maximum value. First we need to get the derivative of function:

<img src="https://latex.codecogs.com/svg.image?f'(x)&space;=&space;\frac{a_2b_2}{(b_2&plus;x)^2}&space;-&space;&space;\frac{a_1b_1}{(b_1-x)^2}" title="f'(x) = \frac{a_2b_2}{(b_2+x)^2} - \frac{a_1b_1}{(b_1-x)^2}" />

When the derivative function is 0, the function has a maximum/minimum value, and we can set some conditions to ignore the solution at the minimum. It is possible to solve

<img src="https://latex.codecogs.com/svg.image?\frac{a_2b_2}{(b_2&plus;x)^2}&space;-&space;&space;\frac{a_1b_1}{(b_1-x)^2}&space;=&space;0&space;" title="\frac{a_2b_2}{(b_2+x)^2} - \frac{a_1b_1}{(b_1-x)^2} = 0 " />

<img src="https://latex.codecogs.com/svg.image?(a_1b_1-a_2b_2)x^2&space;&plus;&space;2b_1b_2(a_1&plus;a_2)x&space;&plus;&space;b_1b_2(a_1b_2&space;-&space;a_2b_1)&space;=&space;0&space;" title="(a_1b_1-a_2b_2)x^2 + 2b_1b_2(a_1+a_2)x + b_1b_2(a_1b_2 - a_2b_1) = 0 " />

Let：

<img src="https://latex.codecogs.com/svg.image?\begin{cases}a&space;=&space;a_1b_1&space;-&space;a_2b_2&space;\\b&space;=&space;2b_1b_2(a_1&space;&plus;&space;a_2)&space;\\c&space;=&space;b_1b_2(a_1b_2&space;-&space;a_2b_1)\end{cases}&space;" title="\begin{cases}a = a_1b_1 - a_2b_2 \\b = 2b_1b_2(a_1 + a_2) \\c = b_1b_2(a_1b_2 - a_2b_1)\end{cases} " />

The previous equation is reduced to a general quadratic equation:

<img src="https://latex.codecogs.com/svg.image?ax^2&plus;bx&plus;c=0&space;" title="ax^2+bx+c=0 " />

We can get the solution:

<img src="https://latex.codecogs.com/svg.image?\begin{cases}x=\displaystyle&space;\frac{-b&space;\pm&space;\sqrt{b^2-4ac}}{2a}&space;\\0&space;<&space;x&space;<&space;b_1&space;\\x&space;<&space;b_2\end{cases}" title="\begin{cases}x=\displaystyle \frac{-b \pm \sqrt{b^2-4ac}}{2a} \\0 < x < b_1 \\x < b_2\end{cases}" />

The solution x is the amount we need to borrow from Pair0.

# Considerations, Reflections & Imporvement Proposals

## Using cenralised exchanges and Aave
I think, it would have been using a CEX (Centralized EXchange) like Binance or Coinbase as off-chain oracle to get price feeds.This is much easier and IMO much more efficent as some of the centralised exchanges have incredible APIs that are getting updated by the second Then to use Avee (lending platform) flashloans (similar to how flashswaps works) to arbitrage whatever DEX (Decentralized EXchanges) I wished. The only 'drawback' that I can see with this approach is that you must pay back the flashloan in the same asset you borrow so you probably need and extra trade.

## Using Flashbots MEV (Miner Extractable Value)
