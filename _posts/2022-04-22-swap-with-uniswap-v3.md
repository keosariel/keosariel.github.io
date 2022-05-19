---
published: true
layout: post
title: Swap with Uniswap V3 (defi)
---

If you're not familiar with [Uniswap](https://uniswap.exchange/), it's a decentralized exchange (DEX) relying on external liquidity providers that can add tokens to smart contract pools and users can trade those directly either through their web interface or smart contracts.

However, we can only trade ERC-20 tokens. For each token there is its own smart contract and liquidity pool. Uniswap, being fully decentralized, no one controls what tokens can be added. If a token pair does not exists anyone can create a pool using their factory, anyone can also provide liquidity. A fee of 0.3% for each trade is given to those liquidity providers as incentive.

### Integrating Uniswap V3
-----
Uniswap can be easily integrated into your smart contracts, with uniswap you can accept multiple tokens as payment and convert them to a different token, for example taking ETH as payment and converting is to USDC.

Importing [Uniswap v3 interface](https://github.com/Uniswap/v3-periphery/blob/main/contracts/interfaces/ISwapRouter.sol) 

```solidity
import "@uniswap/v3-periphery/contracts/interfaces/ISwapRouter.sol";
```

Initializing the router from the contract [address](https://docs.uniswap.org/protocol/reference/deployments) on `kovan` [testnet](https://kovan.etherscan.io/address/0xE592427A0AEce92De3Edee1F18E0157C05861564#readContract). Also you have multiple of the same ERC-20 token deployed multiple times of the testnet, I had to use the [Uniswap app on kovan testnet](https://app.uniswap.org/#/vote?chain=kovan) to get the correct ERC-20 tokens used by the contracts.

```solidity
constructor () {
    uniswapRouter = IUniswapRouter(0xE592427A0AEce92De3Edee1F18E0157C05861564);
    tokens["DAI"]  = 0x4F96Fe3b7A6Cf9725f59d353F723c1bDb64CA6Aa;
    tokens["WETH"] = 0xd0A1E359811322d97991E03f863a0C30C2cF029C;
    tokens["USDC"] = 0xb7a4F3E9097C08dA09517b5aB877F7a917224ede;
}
```

#### Swapping tokens 

```solidity
function swapToken(
    string memory _tokenOut, 
    uint _amountOut
) external payable tokenExists(_tokenOut)

{
    require(_amountOut > 0, "invalid amountOut");
    require(msg.value > 0, "eth sent must be > 0");

    ISwapRouter.ExactOutputSingleParams memory params = ISwapRouter.ExactOutputSingleParams({
        tokenIn  : tokens["WETH"],
        tokenOut : tokens[_tokenOut],
        fee: 3000,
        recipient: msg.sender,
        deadline : block.timestamp + 15,
        amountOut: _amountOut,
        amountInMaximum  : msg.value,
        sqrtPriceLimitX96: 0
    });
    
    uint amountIn = uniswapRouter.exactOutputSingle{ value: msg.value }(params);

    // Refund remaining ETH to sender
    if (amountIn < msg.value) {
        (bool success,) = msg.sender.call{ value: address(this).balance }("");
        require(success, "refund failed");
    }
}
```
**SwapRouter** 

The **`SwapRouter`** will be a wrapper contract provided by Uniswap that has several safety mechanisms and convenience functions. You can instantiate it using ISwapRouter(`contract address`) for any main or testnet. 

**WETH**

In Uniswap there are no more direct ETH pairs, all ETH must be converted to WETH (which is ETH wrapped as ERC-20) first. In our case this is done by the router.

**exactOutputSingle**

This function can be used put in `WETH` and receive and exact `_amountOut` of the out token for it. Later any leftover ETH will be sent back to the sender, this is because uniswap doesn't refund it automatically. If it'not refunded the eth would remain in the contract.

**Fee**

Note that fee is in hundredths of basis points (e.g. the fee for a pool at the 0.3% tier is 3000; the fee for a pool at the 0.01% tier is 100). It's a non-stable, but popular pair, so the fee we are using here is 0.3% (see fee section above).

**sqrtPriceLimitX96**

This can be used to determine limits on the pool prices which cannot be exceeded by the swap. If you set it to 0, it's ignored.

### Working example

I've got the full [code on github](https://github.com/keosariel/lil-defi/blob/master/contracts/LilSwap.sol). It allows you to trade `WETH` for `DAI`, `SAI`, and `USDC` on Kovan testnet.


