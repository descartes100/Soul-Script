---
description: First published on Medium on Mar 28, 2022
---

# Other Possibilities of Smart Contracts: Insights from DLCs

Recently, I delved into Bitcoin's native Discreet Log Contracts (DLC), which challenged some of my rigid notions about the forms that smart contracts can take. Like many others who began their blockchain journey with Ethereum, I found myself harboring similar limited views. Therefore, I felt compelled to write this article to discuss these insights.

### Decentralization = Logic on-chain?&#x20;

Since my introduction to smart contracts, I've been repeatedly told that all logic of smart contracts is public and visible to everyone. My initial reaction was positive: this openness signifies that smart contracts are inherently open-source. Logically, I assumed that this transparency equates to decentralization because once the logic is public, the project developers cannot alter it arbitrarily, and users can verify the code to ensure that there are no malpractices. So far, everything seemed to make sense.

However, DLCs introduced me to another possibility.

Previously, Bitcoin holders looking to earn passive income had two main options:

1. Deposit BTC in centralized Bitcoin banks (e.g., Blockfi, Celsius);
2. Convert BTC to WBTC (Wrapped Bitcoin) and participate in lending protocols on other blockchains.

The first option involves a loss of control and the risk of bank insolvency; the second relies on centralized custody of WBTC and the inherent risks of smart contract vulnerabilities. Both paths depend on third-party trust, contrary to Bitcoin's principle of minimal trust.

Unfortunately, given Bitcoin's limited functionality to just transfers, it seemed impossible to create a native lending protocol on Bitcoin.

This is where breaking conventional thinking is necessary. Apart from lending, what other methods are there to earn passive income? One such method is creating a **covered call**.

A covered call involves holding the underlying asset while selling a call option. In our scenario, this would play out in two ways:

1. If the call is out-of-the-money (OTM) at expiration (i.e., the BTC price is lower than the strike price), the call holder will not exercise the option. The BTC holder retains their BTC and earns the call price as passive income.
2. If the call is in-the-money (ITM) at expiration (i.e., the BTC price is higher than the strike price), the call holder will exercise the option. The BTC holder retains a proportion of their original BTC (strike price/current price) and earns the call price as passive income.

[Atomic.Finance](https://atomic.finance/) utilizes DLC to match buyers and sellers of options, enabling Bitcoin holders to earn passive income without relying on any third party. Specifically, it works as follows:

1. A multisignature address is created on the blockchain, where user A deposits collateral BTC and market maker B deposits money to buy the call.
2. Off-chain, A and B use DLC to co-create two transaction signatures for the oracle states corresponding to the OTM and ITM scenarios:
   * Transaction Signature 1: Oracle (call OTM) + A's signature + B's signature => transfer all BTC in the multisig contract to A
   * Transaction Signature 2: Oracle (call ITM) + A's signature + B's signature => transfer part of the BTC in the multisig contract to A and the rest to B
3. On the settlement date, only one of these transactions will have a valid signature. The oracle will trigger the valid transaction.

Using this method, users maintain complete control over their transaction information. Unlike Ethereum, where all smart contract logic is made public on the network, DLC ensures that most of the smart contract logic occurs off-chain. More accurately, the logic-building process happens off-chain. To outsiders not involved in the transaction, it's unclear that a multisignature address is conducting an options trade until settlement. Afterward, anyone can verify that the transaction was indeed an options trade.

This brings us back to the initial question: Does decentralization necessarily require logic to be on-chain? The answer seems to be no.

### What Are We Decentralizing?

Recently, I read an enlightening article by [Nick Szabo](https://en.wikipedia.org/wiki/Nick\_Szabo), written in 2001 titled "[Trusted Third Parties are Security Holes](https://nakamotoinstitute.org/trusted-third-parties/)" which allowed me to reconsider the concept of "decentralization."

One particular point stood out: Introducing a "trusted third party" (TTP) to resolve trust and security issues in protocols essentially uses traditional security methods to address an unresolved security problem. Essentially, involving TTP doesn't fundamentally solve the security issue; it merely transforms one security problem into another.

What blockchain technology does is eliminate the need for third parties, using the design of protocols to address security issues.

Before delving deeper, let me clarify what "traditional security methods" are and why proponents of decentralization are adamant about removing trusted third parties.

In my view, "traditional security methods" comprise a collection of modern governance concepts and tools, including law, regulation, and policy. In simple terms, these methods involve opening up all activities to legal and regulatory intervention. When disputes arise, laws and regulations act as arbiters of justice, providing a fair resolution. These traditional methods transform the question from 'who is right and who is wrong between interacting parties' to 'who does a trusted third party believe is right or wrong'. In the traditional world, these TTPs are essentially the authorities.&#x20;

At this point, one might realize some peculiar aspects of the traditional world: issues of right and wrong subtly become matters of perspective, and the orientation of these perspectives directly stems from the extent of power. Thus, while humanity has ostensibly built a modern civilized society, at its core, the law of the jungle, where might makes right, still prevails.

This perspective may seem somewhat radical and extreme. From the standpoint of civilizational progress, the emergence of concepts like law, regulation, and government is undoubtedly progressive. Relying on laws that represent a consensus and cannot be arbitrarily altered is certainly better than depending on the whims of someone. However, traditional solutions do not fundamentally resolve security issues. The decisions made by "trusted third parties" are only approximations of "just decisions." Often, this "third-party" role, which can arbitrarily step into any activity, tends to be a source of wrongdoing.

So, how should security issues be addressed?

Blockchain technology offers an alternative solution.

Rather than relying on external parties for "fair judgment," blockchain technology requires the interacting parties themselves to take responsibility for their security. You need to understand what your signature means, and ultimately, only you are accountable for it.

This represents the wisdom of returning to simplicity. When adding conditions worsens the problem, it might be worth trying to remove them.

### P versus NP&#x20;

From Part 2, we learned that decentralization fundamentally assigns the responsibility of security to the users themselves, effectively blocking third-party intervention. Users are capable of engaging in "trustless" interactions with others because blockchain technology provides the means to verify the logic of these interactions.&#x20;

Indeed, the primary purpose of blockchain technology is "verification of logic," not "construction of logic."&#x20;

In a sense, using blockchain technology to "construct logic" constitutes a significant waste of resources. The relationship between verifying and constructing logic is a classic issue in computer science, known as the "[P versus NP problem](https://en.wikipedia.org/wiki/P\_versus\_NP\_problem)." This problem questions whether every problem whose solution can be quickly verified also has a solution that can be quickly found.&#x20;

However, today my focus is not to delve into the long-standing and unresolved answer to this question. Instead, I will analyze the approaches to handling logic inherent in Ethereum and Bitcoin from the perspective of the P versus NP problem.&#x20;

Ethereum makes all the logic of smart contracts public on the blockchain, using the process of solving problems to verify the correctness of the logic. Essentially, this represents a reverse logic of "P versus NP": problems that can be solved are quickly verifiable. This is an uncontroversial truism in the scientific community.&#x20;

Bitcoin, on the other hand, only puts the verification part of smart contract logic on the blockchain, using verification to prove that a problem has been solved. This directly relates to the "P versus NP" problem itself. Although there is currently no consensus in the scientific community, the prevailing view is that the set of NP problems is larger than that of P problems, meaning that the set of solvable problems is a subset of the verifiable problems. From this perspective, Bitcoin addresses a broader range of issues than Ethereum.&#x20;

This conclusion is contrary to the mainstream view. Could it be that Ethereum's much-praised capability to build logic with smart contracts is actually a source of its limitations?&#x20;

I believe it is.

### Hardware or Software

When discussing the current issues with Ethereum, the high gas fees and network congestion are undisputedly the top two concerns. Therefore, many technological innovations in recent years—such as Layer 2, sidechains, and sharding—have been dedicated to addressing these two problems.

Although each of these solutions has its own unique features, they share a core idea: enhancing Ethereum by adding more logic to give it greater functionality.

This is a natural progression and the most obvious solution: if Ethereum has a greater capacity, then naturally, the gas fees would decrease, and the network would be less congested. If we consider Ethereum as a world computer, these solutions are akin to upgrading its "hardware."

However, perhaps we should consider another perspective: is Ethereum’s limited capacity really just due to insufficient "hardware"? Are there issues on the software side as well?

I believe there are. The fundamental issues with Ethereum lie in its software. Simply expanding capacity is a superficial solution; as capacity increases, so does the number of users, and gas fees and network congestion remain high. Ethereum's root problem is its expansion of the "verification logic" function into "construction logic," which leads to the misuse of on-chain resources. This misuse not only results in visible problems like resource wastage and increased costs but also introduces unexpected security vulnerabilities.

### Units of Logical Encapsulation

Developing on Ethereum is undoubtedly challenging. You cannot predict how others will use your code. You leave some interfaces open, which others can call without your permission. On one hand, this greatly promotes cooperation and innovation; but on the other hand, it also opens up unforeseen risk exposures.

Once, I believed that the hacker attacks on Ethereum were a normal part of the system's self-correction, essential for maintaining its vitality and dynamism. However, now, I am somewhat doubtful.

Ethereum has made construction logic its primary function, leading developers to treat "constructability" as the unit of encapsulation. However, "constructable" does not equal "verifiable." When a series of unexpected "constructable" logics combine, they can exceed the bounds of what is "verifiable," leading to disasters.

Disasters are hard to prevent because the potential combinations of "constructable" logic are infinite. Hackers are motivated to exploit these vulnerabilities, but users often are not.

A more scientific and rational method of building smart contracts would be to replace "constructable" logic combinations with "verifiable" ones. Bitcoin's DLCs (Discreet Log Contracts) are an inspiration in this regard.

### **A Middle Path**

I believe that blockchain technology should be used solely for verifying logic, while the construction of logic should occur off-chain. By doing this, not only can we maximize the utility of precious on-chain resources, but we can also minimize users' on-chain footprints and reduce unnecessary security risks.

The implementation path I am currently envisioning is a middle ground between Bitcoin and Ethereum. Blockchain needs to have smart contract functionality, but this functionality should be solely aimed at verifying logic. Different smart contracts can still interact with each other, but the units of interaction must be "verifiable."

This is also why I believe that there is limitless potential in the new public blockchain space. Ethereum and Bitcoin are like two points on a coordinate system, with many possible states in between that are worth exploring, contemplating, and uncovering.\
\
