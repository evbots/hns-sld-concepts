# HNS SLD contract with voting


## Overview

A set of contracts and standards already exist to deploy an SLD contract on Ethereum and resolve names via the HIP-5 protocol. The xnHNS project is one great example of this: it provides contracts conforming to the EIP-137 (ENS) and ERC-721 (NFT) standards, similar to the forever domains implementation, and does not make assumptions about the security properties of the TLD. The xnHNS project implements “snitching” - an economic incentive which punishes the TLD owner by distributing the TLD owner’s stake to SLD owners if the TLD owner changes root zone record. However, xnHNS snitching relies on ChainLink oracles to determine if a snitch is legitimate. This document explores SLD voting as an alternative to ChainLink oracles for determining legitimacy of a snitch. Additionally, another economic incentive is proposed here for SLD owners: veto power over a contract upgrade proposal.

### How snitching works

TLD owners could deposit a stake in the SLD contract, 1 ETH for example. If an SLD user comes to believe that TLD records have changed in some undesirable way (HIP-5 record is deleted), they have financial recourse they can pursue through snitching. If a snitch is initiated and then confirmed, all SLD users could receive an equal part payout of the TLD stake.

There are many ways to modify this scheme, but that is the basic gist.

### Potential Problems with ChainLink

There are two models for ChainLink nodes to report values:
- DDM (decentralized data model) - nodes report data on-chain and a smart contract aggregates results and reports a single value.
- OCR (off-chain reporting protocol) - nodes p2p aggregate values and report a single value on-chain.

DDM is currently open source and available for others to copy and use. OCR is not open source and is only used by chainlinik data feeds so far. Right now, the only thing stopping a ChainLink node from reporting bad data into a custom SLD contract is their reputation.

To get ChainLink working today in an SLD contract, you recruit ChainLink node operators, convince them to install your adapter on their nodes, and hope that they correctly use it and don’t incorrectly modify it before reporting values into your smart contract. There are multiple ways this could fail.

First is an honest mistake: ChainLink node operators could be looking at old HNS blockchain state by accident, and report a stale value. They could modify the adapter they install in order to relay data from HNS to ETH, which results in buggy values. And second is by malicious intent. They could register some SLD names, and then report false values in order to receive the stake.

If the context here was a price feed, you could cull faulty node operators from your oracle network, but in this case, because values are reported so infrequently, you don’t have the opportunity until something bad has happened.

### Voting as an Alternative

Instead of relying on ChainLink, a snitch could be verified against a naughty TLD owner by voting which reaches a threshold, say 75%. If the threshold is met, a snitch is confirmed and the TLD owner’s stake is distributed equally to SLD owners.


### Naive Voting Implementation

The naive approach to SLD voting would allow every SLD exactly one vote. This has many problems, but the most important problem is attacks. If you can quickly and cheaply acquire SLDs by registering them or on secondary markets, and then immediately use them for voting, your protocol can be attacked. This problem is exacerbated by NFT lending marketplaces, where SLDs could be borrowed for a short time period, used to vote, and then returned.


### Proposal for Better Voting Implementation

I propose requiring a lockup and cooldown period for SLD owners, in order to qualify for a vote. Practically, this means that an SLD would have to exist at an address for some time period, one year for example, and after a successful vote, would not allow a transfer out of that address for some time period, say one year. In this example, SLD owners would have to be locked in to SLD ownership for a period of 2 years. This means capital as well as time is spent on voting, as opposed to just capital.

In order to build this, you could modify the transfer method to fail if a vote has taken place in the last n blocks, and the vote method would not allow voting on the part of an SLD owner if that SLD has not existed at an address for at least n blocks.

You could also extend this to other standards such as the Consumable Extension (EIP-4400). changeConsumer could not be called according to the same rules as transfer, identified above.


#### Benefits to this proposal

- SLD owners can still have liquidity, aka approving listings to opensea, while still accruing voting rights in their SLDs.
- Any transfer associated with a sale of the domain (such as a sale on OpenSea), the voting eligibility clock will automatically reset.
- You have to lock up capital for some time period. This means that there is a real opportunity cost to attacking the protocol.


#### Problems with this proposal

- Account abstraction is still a big problem here: users can create synthetic contracts, such as their own NFT contract, and transfer ownership of the SLD into that contract, and then mint synthetic NFTs and trade those on other markets. This would effectively side-step the voting protections. In general, I think that this is a bit far fetched, but could become a legitimate problem if the protocol grows in popularity (over 100k registrations).
- Any transfer of the SLD will reset the clock on if that SLD is eligible to vote. There are many legitimate reasons that a user would want to transfer ownership of an SLD between their own addresses, so this is a problem.


### Vetoing Upgrades

SLD voting could also be leveraged to signal approval or disapproval for a TLD’s owners attempt to upgrade the SLD contract. This is achieved by controlling the TLD owner’s ability to withdraw their stake. If the stake is large enough, TLD owners will want to be able to withdraw the stake in order to deposit it into the new upgraded contract.

This could work by a TLD owner attempting to withdraw, but being subject to an N blocks waiting period. During that period, SLD owners have the chance to reach some threshold (say 75%) in order to veto the TLD owner’s ability to withdraw their stake to another contract. If the veto is successful, the owner’s stake is stuck until they propose an upgrade that the majority of SLD owners approve of.
