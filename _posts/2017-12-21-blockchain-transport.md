---
title: Blockchain Transport
date: 2017-12-21 07:00
lastmod: 2017-12-21 07:00
description: Introducing The Planar Network, a platform for integrated multi-modal travel tickets based on blockchain technology.
layout: post
---

A few months ago I gave a [high level overview of blockchain technology](http://ljn.io/posts/blockchain-introduction/) and promised to show how blockchain technology might be useful in the transport industry. In this post I'll introduce The Planar Network, a platform for interoperable multi-modal travel based on blockchain technology.

## Public Transport Networks

Privatisation of public transport networks is commonplace across many parts of the world, but the implementation of the privatisation varies. The UK rail network was privatised in 1993 under the condition that it continued to offer passengers a single ticket that's valid across multiple operators. Other countries, like France, Germany and Poland do not offer interoperable tickets and a ticket must be purchased for travel on each operator's network.

### Interoperable Networks

In order to offer interoperable tickets there must be a common ticket format that all operators adopt. This means there must be a governing body to enforce the standards, usually by an expensive and bureaucratic accreditation process.

Tickets sold that use multiple operators must go through a settlement process to divide the money and allocate it to each operator appropriately. This is adds a significant amount of time to an already slow financial process and is a potential source of discrepancies. Typically there is no single source of truth for tickets sold, tickets created, tickets used and money settled. Ticket retailers need to integrate with an issuer which then needs to report tickets issued to the settlement system. 

Retailers need to in depth knowledge of the ticketing standards and infrastructure in order to start retailing.	

### Isolated Networks

Where there are no interoperable tickets, travel across multiple operators must be arranged separately and results in many transactions that must be managed independently by the passenger.

It is often not possible to offer the passenger a ticket permitting travel across a choice of operators, forcing the passenger to make decisions at the time of purchase rather than the time of travel.

Each operator will typically use their on format of ticket, maintaining it’s own notion of what the types of ticket are and what restrictions apply. This forces the passenger to become an expert of multiple systems to gain the best value from each.

Both of these types of transport network have problems that can be solved using a blockchain.

## Blockchain Technology

As covered in [my previous post](http://ljn.io/posts/blockchain-introduction/), blockchain technology was introduced as a means to perform financial transactions over a peer-to-peer network, removing the need for a central trusted authority. Initially the blockchain was tied to an in-built currency but recently the concept has been expanded to include programmable smart contracts. These smart contracts enable us to model any business transaction in a peer-to-peer network. 

Using a blockchain and smart contracts we can create a platform that integrates isolated transport networks without all the pitfalls of interoperable networks. 

## The Planar Network

The Planar Network is a platform that provides retailers, operators and transport authorities the necessary infrastructure to create and retail interoperable tickets regardless of mode of transport or geography. 

By storing the network topology on the blockchain we can establish a number of trusted transport operators (Operators) that are permitted to offer Fares for transport between a subsection of the overall transport network. Each Operator can be assigned its geographical region by a transport authority (Authority). There may be many Authorities on the platform covering a wide range of modes and geographical regions.

![topology](/asset/img/blockchain-transport/topology.png)

### Ticket Creation

After the Operators have setup their Fares a ticket retailer (Retailer) can execute a smart contract that turns a Fare into a Ticket stored on the blockchain. 

![simple-contract](/asset/img/blockchain-transport/simple-contract.png)

As the smart contract is executed money is transferred from the Retailer to the Operator instantaneously. The smart contract is executed as an atomic transaction meaning it will either entirely fail or entirely succeed. There is no situation where the Ticket could be created without the necessary money being exchanged. As the blockchain is immutable it is also not possible to go back and change any of the details of a transaction that has taken place.

All Tickets stored on the blockchain belong to a wallet, in the case of a blockchain a wallet is just a public/private key pair that can be used to prove ownership. 

### Interoperable Tickets

Now that a process to create Tickets has been established, we can extend the smart contract to allow multiple Fares from a range of Operators. 

![multi-contract](/asset/img/blockchain-transport/multi-contract.png)

The smart contract is still executed as a single transaction but the money from Retailer is split up and sent to each Operator according to the Fare they've provided. The details of all the Fares are stored in the Ticket on the blockchain. 

### Settlement

Although many blockchains come with an in-built cryptocurrency it is not necessary to use that as the basis for transactions. It is quite common to digitally represent [fiat currencies](https://en.wikipedia.org/wiki/Fiat_money) to eliminate the risk of fluctuating exchange rates. 

For a Retailer to start selling tickets they need to have money in the network to execute the smart contract that creates tickets. The Retailer can  cash-in to the network using an exchange to purchase digital versions of fiat currencies. Operators can take the money they accrue through ticket sales and cash-out to their local currency using the exchange. 

Using digital representations of fiat money allows us to bind the ticket creation to the financial settlement. Where there are multiple Operators with different fiat currencies the Retailer would need to possess the requisite amount of each currency in the network before being able to execute the contract. 

The retail experience for the customer is unchanged. The customer does not act directly on the blockchain, instead they act through Retailers' websites or apps using traditional means of transferring money such as card payments.

![exchange](/asset/img/blockchain-transport/exchange.png)

### Trust

Most transport networks have a process companies go through before they can be trusted to retail tickets correctly. This process usually involves some form of accreditation to ensure that tickets are being created correctly and a [financial bond](https://en.wikipedia.org/wiki/Bond_(finance)) to ensure retailers are financially solvent. The process of retailing a ticket often involves creating the ticket so operators and authorities put this process in place to ensure anyone retailing tickets can give them the money they are owed.

In the Planar Network this trust is built into the platform. A Retailer can only create Tickets using the Fares the Operators have created, and they must do it via a smart contract that ensures the Operator receives their money. There is no need for a bond as the Retailer must give the money to the Operator at point of sale.

### Passenger Travel

Now that we've established a means to retail and create tickets we need to allow the passenger to travel using their Ticket. That means integrating with a range of entrenched infrastructure including gates, ticket scanning and validation mechanisms. It’s not feasible to immediately upgrade the existing infrastructure to directly query the blockchain for ticket validation. Instead, an integration layer must be provided to create ticket coupons in each network's existing formats to ensure that existing infrastructure will continue to work.

By adding a number of ticket distributors (Distributors) to the network we can implement a collection smart contract that turns a blockchain Ticket into one of networks native formats.

![barcode-collection](/asset/img/blockchain-transport/barcode-collection.png)

Introducing a Ticket collection contract decouples ticket retailing and ticket distribution allowing the passenger to choose their ticket format up to the point of travel, rather than at time of purchase. It also removes the need for Retailers to understand the inner works of each ticket format. 

Where the passenger is travelling across multiple operators with different formats the Ticket can be part collected by multiple Distributors. Each Distributor providing travel coupons (native tickets) for the parts of the journey it understands. 

As more transport systems become integrated their ticketing formats will become more homogeneous meaning fewer ticket Distributors will be required. As adoption of the blockchain increases more validation infrastructure will integrate directly with the blockchain and the role of the Distributor can be phased out entirely. 

## Benefits

By using smart contracts to create tickets in a controlled manner collapses a multi-stage process of ticket retailing, creation, settlement and reporting down to a single transaction, removing a whole class of errors in the process. 

We've moved from a system of trust that's bought via accreditation and a bond to a system of trust that's built into the platform. In the process of doing that we've also created a much more even distribution of responsibilities. 

The role of the Retailer is now focused around the customer. They no longer have to know the details of multiple ticket standards in order to retail tickets. They cannot be held accountable for creating tickets incorrectly because they can only create tickets from fares created by the operators. 

The barrier for entry to market is much lower as Retailers don't need to understand the details of every ticket format and there is much less need for an accreditation process and financial bonds.

Providing access to more operators from different regions or modes of transport gives retailers access to markets they couldn't enter before.

The transport authority is now focused on defining the rules of the network rather than enforcing them. The cost of operating the IT infrastructure is shared between all the network participants. 

Operators get instant access to ticket revenue and a more accurate settlement process. They're no longer suffering penalty for mistakes made by retailers during the ticket creation or settlement process.

Passengers can purchase tickets for travel across multiple transport operators in a single transaction, from a single retailer. 

There is nothing to prevent a single integrated ticket being created for a trip involving air, bus, ferry and rail travel. If the authorities in each of those sectors can reach an agreement for an integrated fare it could be sold by any retailer connected to the network. Even without a cross-authority agreement in place multiple tickets could be packaged into a single transaction giving passengers access to a global transport network.

Having a portable ticket wallet that can store tickets for travel across multiple operators greatly simplifies the practicalities of traveling across a multi-operator network. All tickets can be accessed via a single app and be collected using the networks existing infrastructure.

The passenger doesn’t have to choose what format the tickets coupons are in before they purchase, the ticket can live in their digital wallet until they collect them as a native format.

## Further Reading

The Planar Network is still in early stage development but I've received a lot of positive feedback from people in the industry so I'm keen to get feedback from a wider audience. If you have any thoughts or suggestions [please let me know](mailto:linusnorton@gmail.com). After developing a proof of concept I'm hoping to pilot with three transport companies that are not currently integrated. 

If you want to read more about the proposal you can read the [long form](/asset/pdf/planar-network.pdf).
