---
title: Introduction to Blockchains
date: 2017-09-29 06:00
lastmod: 2017-09-29 06:00
description: A high level overview of blockchain technology covering what it is, how it works and what it's practical applications are.
layout: post
---

In this post we're going to cover what a blockchain is, how it relates to cryptocurrencies, how it works and what it's practical uses are. Each one of those topics is worth a post in itself so we'll be taking a fairly high level view but it should be enough to get a feel for this emerging technology.

## Bitcoin

To explain what a blockchain is we need to examine how they became popularised and for that we need to talk about bitcoin. Bitcoin is a peer-to-peer cryptocurrency introduced by Satoshi Nakamoto in 2007. It was not the first digital currency to be released but it was the first to gain mainstream traction because it solved a very difficult distributed systems problem: the double spending problem.

In any distributed system with a large number of nodes there is a communication lag across the network, so reaching any sort of consensus can take time. For a digital currency this is a critical issue because one node might mischievously send two transactions to different nodes at the same time and have them both accepted, effectively spending their money twice. Existing consensus algorithms used for relational database are not an option here as they are designed for a small number of trusted nodes. In a peer-to-peer network those algorithms would take too long as they are typically as slow as the slowest node on the network.

Bitcoin's solution to this problem was to write down every transaction in a distributed ledger that is shared across all the nodes in the network. The ledger is immutable so no one go can back and change it. 

This distributed ledger is the blockchain.

The ledger contains a number of pages called blocks that each contain a number of transactions. Each transaction is a hash of the preceding transaction and the recipient's public key signed using the sender's private key.

![transaction-hash](/asset/img/blockchain-introduction/transaction-hash.jpg)

Hashing each transaction using the previous transaction ensures that the ordering of those transactions cannot be changed. This in turn ensures that money cannot be double spent as anyone with access to the blockchain can easily verify whether the sender still has the money to send.

A blockchain is a distributed ledger of transactions that have been ordered using a cryptographic hash. 

## Consensus algorithms

Now that we have a mechanism for transactions to be stored in an ordered manner we need a way for everyone in the network to agree what the correct ordering is. 

This is a called a consensus algorithm.

There are many types of consensus algorithm, the one a bitcoin uses is called a Proof of Work algorithm. The principle is that transactions are organised into blocks that are similar to the pages in the book of a real world ledger. In order to create a block of transactions you must do some computationally difficult "Work", and then it's a case of the longest blockchain wins. 

This may not sound like an intuitive way to reach consensus until you understand the problem it's trying to solve. The problem is commonly referred to as the [Byzantine Generals Problem](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance) - several generals are forced with a decision to either attack or retreat. Success of either depends on everyone agreeing to the same action, the problem is that some generals are spies and may attempt to subvert the voting process by sending different votes to other generals.

![generals](/asset/img/blockchain-introduction/byzantine-generals.jpg)

To put this in blockchain terms, a member of the network may try to put forth an alternate version of the truth. By making the blocks in the blockchain computationally difficult to create we ensure that as long as the well behaved members of the network have more CPU power than the bad ones then the longest chain will always be the most correct one.

An essential property of the Proof of Work is that it is hard to generate but easy to verify - in other words, a [P!=NP](https://en.wikipedia.org/wiki/P_versus_NP_problem) problem.

Bitcoin's proof of work algorithm increments a number until the hash of that number equals 0. Trying to find these numbers to create new blocks is known as mining. In the bitcoin blockchain each new block comes with a reward to incentivize members to verify transactions and progress the blockchain.

## Smart contracts

In the bitcoin network the blockchain acts as storage for financial transactions where money is moved between two parties. Recently there have been a number of new [cryptocurrencies](https://ethereum.org/) and [blockchains](https://www.hyperledger.org/projects/fabric) that have broadened the scope of what a transaction is. Rather than being limited to one type of transaction, programmable logic known as a smart contract is executed when the transaction is performed.

In a network that uses smart contracts the role of the blockchain is the same - it stores an ordered list of transactions but the transactions are state transitions that occur as a result of executing the contracts. 

## Blockchain without the cryptocurrency

With the introduction of smart contracts transactions are not just about moving money. This means it's entirely possible to have a blockchain without a cryptocurrency. Consequently this means that you might need a different way to incentive nodes to verify transactions, and an entirely different consensus algorithm. This is a rapidly evolving area and approaches vary.

If you remove the cryptocurrency from a blockchain you are essentially left with an event sourced, distributed database that stores a globally shared world state. 

The distributed manner of the database allows network participants to execute contracts with each other, without the need for a central authority or centralised infrastructure. 

## Use cases

Blockchain technology is often described as a solution without a problem, and there is an element of truth in that. Aside from bitcoin for financial transactions and ICOs for fund raising there have been a limited number of mainstream applications for blockchain technology. But it's worth remembering that blockchains are just another type of storage and over time I expect that using a blockchain will be similar to choosing whether to use a relational database or a document store or a queue. They all have their strengths but if you really want you can usually get the job done with either of them. 

Right now blockchain technology is new and relatively difficult to work with mean people are tending to stick with more mainstream options.

That said, there are obviously cases where blockchain technology excels, it often comes down to trust. In a scenario where you do not want to rely on a central authority to provide centralised infrastructure or control all the interactions on the network then a blockchain is a good choice. It allows two parties to cooperate without the need for a mutually trusted middleman.  

The introduction of blockchain technology is following a similar path to the introduction of peer-to-peer file sharing. Peer-to-peer technology allowed users to download files from each other directly without relying on a central server that could be shut down at any point. Drawing an analogy with peer-to-peer technology is risky as it's introduction was fraught with legal problems but the important thing to remember is that while file sharing still has many illegal use cases, the peer-to-peer technology powering it was legitimised. The same has happened with the blockchain - Sweden are currently looking at using a blockchain as their land registry [1](https://qz.com/947064/sweden-is-turning-a-blockchain-powered-land-registry-into-a-reality/), a Russian airline is using it to store tickets [2](https://futurism.com/an-airline-just-started-using-ethereum-blockchain-to-issue-tickets/) and there are many uses for chain of custody [3](https://scm.ncsu.edu/blog/2017/03/10/can-blockchain-become-the-solution-for-anti-counterfeiting-and-chain-of-custody/).

In next month's blog post I will introduce a system that enables multi-modal transport across operators from different networks using a blockchain.

## Further reading

I would highly recommend reading the [original paper on Bitcoin](https://bitcoin.org/bitcoin.pdf) it is both very short and easy to digest. 

There is also a very good overview of [consensus algorithms](https://www.persistent.com/wp-content/uploads/2017/04/WP-Understanding-Blockchain-Consensus-Models.pdf) provided by [persistent.com](https://www.persistent.com).


