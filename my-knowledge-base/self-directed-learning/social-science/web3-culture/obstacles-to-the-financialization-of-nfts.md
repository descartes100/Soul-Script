---
description: >-
  -Wash Trading and Money Laundering Concerns, First published on Medium on Mar
  15, 2022
---

# Obstacles to the Financialization of NFTs

This article was inspired by a discussion in the Eth Research forum titled "[How does the value of an NFT is calculated as a loaning asset?](https://ethresear.ch/t/how-does-the-value-of-an-nft-is-calculated-as-a-loaning-asset/10514)" The author of the discussion proposed an arbitrage method using NFTs, essentially getting something for nothing. Let’s explore this arbitrage process.

### The Arbitrage Process:

1. User Bob creates an NFT from a random image using his first account, addr1. Initially, this NFT is priced at zero.
2. Bob then lists this NFT for sale at a price 𝑐1
3. Using his second account, addr2, Bob buys the NFT for 𝑐1
4. The NFT is now valued at 𝑐1
5. Bob uses the NFT as collateral for a loan, securing 𝑐1×𝑟 (where 𝑟 is the loan rate for NFTs).
6. As long as 𝑐1×𝑟 exceeds the transaction fees, Bob has an incentive to execute this process.

### Identified Problems in the Arbitrage Process:

1. The rarity of NFT transactions leads to insufficient liquidity, which reduces 𝑟
2. The obvious connection between Bob's two accounts suggests wash trading, which also reduces 𝑟

### Solutions within the Ethereum Ecosystem:

#### Method One: Wash Trading.

Is the NFT not traded enough? The solution could be more transactions.

By repeatedly buying and selling the NFT across multiple accounts—Bob’s third account buys from the second, the fourth buys from the third, and so on—Bob can artificially inflate the price. This is known as wash trading, where the buyer and seller are fundamentally the same entity, creating misleading transaction records.

In traditional markets, wash trading is a tactic typically reserved for large institutions or wealthy individuals, as it requires multiple financial accounts approved through KYC (Know Your Customer) and significant capital to absorb market liquidity. However, on blockchain markets dealing with fungible tokens, the barriers to wash trading are lower, requiring only capital since creating a new account on the blockchain is as simple as a few clicks. In the nonfungible token (NFT) market, even the monetary requirements are minimal since the money can be recycled across accounts, allowing for multiple transactions with the same funds.

However, with due diligence, it's possible to trace and link the money moving between different accounts back to a single individual, a practice that NFT lending institutions are beginning to implement to avoid falling into such traps.

#### Method Two: Money Laundering.

How can one break the connection between accounts established by transfers? Through money laundering.

Using the Tornado Cash Protocol as an example, a user can transfer amounts like 0.1 ETH/1 ETH/10 ETH/100 ETH from account addr1 to Tornado and receive a private key. After an indeterminate amount of time, they can use another account, addr2, enter the private key, and Tornado will transfer the funds to addr2. This process is designed to prevent exposure of the transferor’s identity by using fixed transfer amounts.

This makes it impossible to link addr1 and addr2 as belonging to the same individual. For our NFT arbitrageur, paying double the transaction fees to break the linkage between accounts allows for untraceable wash trading.

With these tactics of wash trading and money laundering, the NFT arbitrage process seems complete. However, one assumption remains unaddressed: Are there actually companies that provide collateral loans for NFTs?

A quick search reveals a few such companies, but these firms currently only lend against NFTs that have received broad market recognition. Thus, the assumption of using a random image as collateral falls through. Therefore, the discussed NFT arbitrage mechanism remains hypothetical and not yet practicable.

<figure><img src="https://lh7-us.googleusercontent.com/zyxvvB7xutz8W4mUE4i5NAPejpMVqf0P_5_-EePf69RCzKBvnWeo_TgVVHK-Wt63_KaSDuEeHFYvPZmFOGgFMsL_yZDNqiGUu6ecgG_4peuZOhBylWRbotMn1A3dLV7DyPOvJ5am9zXgRURsurUWf_Y" alt=""><figcaption></figcaption></figure>

Nevertheless, this thought experiment is not without merit. It highlights the challenges of financializing NFTs: creating fake transactions is low-cost and risk-free. Digging deeper, the reason creating fake transactions is so cost-effective and risk-free is due to their nonfungible nature, combined with the powerful privacy tools of blockchain technology.

Recalling a conversation with a friend about creating a financial instrument to short NFTs, it was initially just a fun idea without exploring its feasibility. Now, it seems quite challenging because the very nature of NFTs makes them difficult to financialize, let alone short.

\
