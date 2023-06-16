# DvP Tutorial 

This is a Navigator demo which demonstrates a few Canton concepts.

## What is a Canton Domain? 

A Canton domain is a completely sovereign application with its own 
- business rules, 
- privacy policies, 
- asset classes, 
- Entitlements, 
- Datamodels, 
- APIs, 
- etc.. 

There are three domains in this applications in this tutorial, but see this [^1] about an important caveat. 

[^1]: This tutorial does not in fact have three domains. But this makes no functional difference for the features we're highlighting here. To make this demo truly multi-domain, the Cash domain would deploy `Cash.daml` as its module, the Security domain would deploy `Security.daml`, and of course the trading domain would deploy `Trading/*` as its module. All that's required is that the Trading domain be aware of the Cash and Security domains' modules.

### Cash domain 

This domain, spelled out in `Cash.daml`, allows a few features associated with a liquid cash instrument. Firstly, cash holdings can be split up, transferred, recombined, etc.. Notably, a cash holding may be partially escrowed for the benefit of a third party. 

### Security domain 

This domain is spelled out in `Security.daml`, and allows for the issuance, transfer, and splitting of a security holding. Also, it spells out the concept of encumberance and unencumberance. 

### Trading domain 

This is spelled out in `Trading/Model.daml` and `Trading/Service.daml`. The idea is that security holders can offer trades to cash holders at a certain price. Accepted deals then cause a DvP between the buyer and the seller. This domain first locks the asset when it is offered as part of a trade, and locks the cash when the buyer accepts that offer. A single DvP transaction then swaps the asset. 

## What is a Canton Node? 

All participants interact with Canton via a node. Each node may connect to multiple domains, and is subject to the sovereignty of those domains. For the purpose of this tutorial, there are four participant nodes: 

### Custodian Node 

This node only connects to the Security domain, and is responsible for facilitating the transfer of assets. They play the role of the `custodian` in Securities contract. 

### Bank Node 

This node only connects to the Cash domain, and is responsible for the liability of various cash holdings. They play the role of the `bank` on Cash holdings.

### Buyer and Seller Nodes 

These two nodes connect to all three Domains and are entitled very similarly, except on initial setup the buyer has cash and the seller has securities. 

## What is a DvP on Canton? 

For the purpose of this tutorial, a DvP occurs if and only the following occur atomically: 
- Cash moves from the buyer to the seller. In particular, that cash moves 
  - FROM: Held by buyer, but escrowed for the benefit of Seller 
  - TO: Out of escrow and held by the Seller.
- Securities move from the seller to the buyer. In particular, securities move
  - FROM: Encumbered by the seller for the benefit of the buyer 
  - TO: Unencumbered and owned by the buyer. 

By Atomically, we mean that all of the above happens, or none of the above happens. For example, if by some chance (e.g. through the `DivineIntervention` choice on a cash holding) the cash moves out of escrow before the DvP, then the transaction cannot happen. 

An important feature of this DvP is the privacy of the participants involved. 

## What do we mean by Privacy? 

Privacy means that participants can only see business contracts that is relevant to them. By see, we mean that contracts that are not relevant to them do not appear on their node at all. 

So after a DvP is completed, each participant will have seen:

| Item                                                    | Buyer        | Seller       | Bank         | Custodian |
|---------------------------------------------------------|--------------|--------------|--------------|-----------|
| Buyer's Total Cash                                      | :green_book: |              | :green_book: |           |
| Buyer's Escrowed Cash (Seller's Benefit)                | :green_book: | :green_book: | :green_book: |           |
| Seller's Cash from Escrow (Not escrowed)                | :eyes:       | :green_book: | :green_book: |           |


Key: 

- :green_book: : Is on the ledger at that node, in the Cash Domain 
- :blue_book:  : Is on the ledger at that node, in the Securities Domain 
- :ledger:     : Is on the ledger at that node, in the Trading Domain
- :eyes:       : The node holder is aware of the existence of the contract but does not have it on their ledger. 