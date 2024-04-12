# 2024-Spring-HW2

Please complete the report problem below:

## Problem 1
Provide your profitable path, the amountIn, amountOut value for each swap, and your final reward (your tokenB balance).

> Solution

## Problem 2
What is slippage in AMM, and how does Uniswap V2 address this issue? Please illustrate with a function as an example.

### Solution
Slippage in AMM refers to the price change that can occur between the time a trade is submitted and when it is executed. Because the AMM use a formula to automatically determine the price of a token based on its supply and demand, when a large trade is made, it can significantly impact the token price, leading to slippage.
Uniswap V2 addresses this issue by allowing users to set a limit on the amount of slippage they are willing to tolerate for a trade (`slippage tolerance`).

```python
function swapTokens(
    uint256 amountIn, 
    uint256 amountOutMin, 
    address[] calldata path, 
    address to, 
    uint256 deadline
) external returns (uint256[] memory amounts) {
    IERC20(path[0]).transferFrom(msg.sender, address(this), amountIn);
    IERC20(path[0]).approve(address(uniswap), amountIn);

    // Perform the swap
    amounts = uniswap.swapExactTokensForTokens(
        amountIn, 
        amountOutMin, 
        path, 
        to, 
        deadline
    );
}
```
In this function, if the actual amount of output tokens is less than `amountOutMin` (set by the owner), `swapExactTokensForTokens` will revert.

## Problem 3
Please examine the mint function in the UniswapV2Pair contract. Upon initial liquidity minting, a minimum liquidity is subtracted. What is the rationale behind this design?

### Solution
The mint function in the `UniswapV2Pair` contract is used to add liquidity to the pool. Upon initial liquidity minting, the function burns (or subtracts) the `MINIMUM_LIQUIDITY` (` _mint(to, MINIMUM_LIQUIDITY);`). This is done to prevent rounding errors in the future.

## Problem 4
Investigate the minting function in the UniswapV2Pair contract. When depositing tokens (not for the first time), liquidity can only be obtained using a specific formula. What is the intention behind this?

### Solution
The intention behind the formula is to ensure that the ratio of the deposited tokens to the existing reserves in the pool remains constant.
For instance, if `amount0` and `amount1` are amounts of 2 tokens being added to the pool, `totalSupply` the current supply and `reserve0` and `reserve1` are the reserves for the two tokens, the formula would look like this:

`liquidity = min(amount0 * totalSupply / reserve0, amount1 * totalSupply / reserve1)`

## Problem 5
What is a sandwich attack, and how might it impact you when initiating a swap?

### Solution
Anyone can see pending transactions on blockchains (`transparency`), therefore they can identify large pending trades that can significantly impact the price of the token being bought or sold (supply-demand). It's done in 3 phases (hence the funny sandwich nickname):
1. Front-running (First slice of the bread): The attacker quickly submits a transaction of their own before the large pending transaction. this transaction involves buying the token the target swap intends to buy. This drives the price of the token up due to increased demand.
2. Meat in the middle: The large swap transaction from the victim executes at the inflated price caused by the attacker's front-running purchase. **This is where I (the victim) feel the impact since I receive less token than I expected due to the higher price (hence, slippage)**. 
3. Back-running (Second slice of the bread): The attacker then executes a second transaction (back-running) where they sell the tokens they just bought at the inflated price, making a sizable profit. The price then falls back to the initial price (almost).

