# Implementing Full Node Incentives

Most Celo mobile phone users will run a [light client](https://github.com/ethereum/wiki/wiki/Light-client-protocol)
that connects to full nodes via a light protocol (*e.g.*,
[Light Ethereum Subprotocol (LES)](https://github.com/ethereum/devp2p/blob/master/caps/les.md)) in order to interact with the
blockchain network. A good user experience for mobile phone users will require the full node netork to be:

* robust and highly available; and
* simple for users to access.

This discussion describes existing approaches for developing a full
node network in light of the robustness and simplicity requirements
and discusses a possible complementary approach that could help
strengthen the network.

## Existing approaches

This section provides an overview of two approaches for building full
node networks explored in the Ethereum community ([Volunteer full
nodes](#volunteer-full-nodes) and [Subscription full
nodes](#subscription-full-nodes), and an approach currently
implemented in Celo for [dontations to full
nodes](#dontations-to-full-nodes).

### Volunteer full nodes

A volunteer-based full node network is composed of full nodes that
handle light protocol requests without requiring payment. Servicing
light protocol requests might consume relatively little computation
compared to processing transactions, so organizations might be willing
to run volunteer full nodes for light clients. For example, the Celo
organization could run a volunteer full nodes to aid the adoption of
Celo; or validators could run a voluneer nodes to help drive the use
of Celo and generate goodwill from potential stakers.

Pros:

* Requires no changes to the LES.
* Simple user experience.

Cons:

* Some evidence exists that the network of volunteer full nodes on the
  Ethereum network is not responsive to spikes in usage (*e.g.*, [_An economic incentive for running Ethereum full nodes_](https://medium.com/vipnode/an-economic-incentive-for-running-ethereum-full-nodes-ecc0c9ebe22)).

### Subscription full nodes

A subscription full node network would charge light clients a
subscription fee to service light protocol requests. A mobile phone
user would select the subscription full node network that they would
like to use and pay the subscription fee. The fee could be collected
periodically ("monthly pass") or held in an escrow contract and
deducted after processing transactions for a light client ("punch
pass"). The [Vipnode](https://vipnode.org) project is exploring this
direction for Ethereum.

Pros:

* Incentavizing full nodes might facilitate higher availability by
  encouraging more full node participation when Celo traffic
  increases.

Cons:

* Potentially requires changes to the LES (*e.g.*, to identify light
  clients as subscribed members).
* Introduces friction into the user experience by requiring a
  subscription service.
* Bootstrapping a new mobile phone user becomes more complicated
  because they would need some Celo currency to sign up for a
  subscription to begin sending light protocol requests (or rely on
  a separate set of potentially less performant volunteer full
  nodes).

### Dontations to full nodes

Celo currently supports donations to full nodes via `GasFeeRecipient`:
each transaction a light client submits includes the etherbase of the
full node to receive a small fee. Because a light client could
potentially announce their transactions directly and include their own
etherbase, we consider setting `GasFeeRecipient` to the etherbase of
the full node a donation. The expectation is that donation amount to
the full node will be low enough compared to the mining transaction
fee that light clients will not be highly incentivized to run their
own custom implementations that set the `GasFeeRecipient` to
themselves, but still high enough to incentivize enough full nodes to
participate in the network.

Pros:

* Facilitates a simple user experience
* Incentavizing full nodes might facilitate higher availability by
  encouraging more full node participation when Celo traffic
  increases.

Cons:

* Requires changes to the LES.
* Challenges around robustness of submitting transactions through full
  nodes: because transactions include an etherbase of the full node in
  `GasFeeRecipient`, light clients must retry submitting transactions,
  or create N separate transactions to submit through N different full
  nodes.
* Might require changes to web3.js ([sendtransaction](https://web3js.readthedocs.io/en/v1.2.1/web3-eth.html#sendtransaction)
  would return multiple `transactionHash` values).
* Donations might not provide enough incentives for to create a highly
  available network of full nodes.

## A complementary approach: reputation tracking full nodes

A reputation tracking full node would collect or request information
about potential clients to understand their past behavior and
willingness to dontate a transaction fee to the full node for servicing
light protocol requests. Full nodes would expect light clients to
include a fee for the full node with each transaction (*e.g.*, the
`GasFeeRecipient` Celo already implements). Reputation tracking full
nodes would share evidence of deviant light client behavior and
prioritize requests for clients that seemed likely to pay the
transaction fee.

As a concrete example, a light client could send a signed message to a
full node confirming their intent to incude a small transaction fee in
their next transaction. That message would look like:

```go
type IntentToPay struct {
        Fee             uint64
        Client          *common.Address
        Recipient       *common.Address
        Signature       *big.Int
}
```

Full nodes would be responsible for identifying deviant behavior of
clients connected to them (*e.g.*, many client requests without a
transaction or clients transacting without paying a fee to the full
node) and sharing the `IntentToPay` via a full node overlay network
(*e.g.*, DHT) to help prove the behavior. Other full nodes
participating in the network could check for evidence of past deviant
behavior.

Pros:

* Facilitates a simple user experience.
* Incentavizing full nodes might facilitate higher availability by
  encouraging more full node participation when Celo traffic
  increases.

Cons:

* Requires changes to the LES.
* Susceptible to light clients with good reputations that decide to or
  are unable to pay addtional transaction fees (but could probably
  bound how frequently each client could consume resources for free).
* Requires full nodes to maintain an overlay network for tracking and
  managing client reputation (e.g., a BitTorrent-like storage
  network).
* Imposes contraints on transactions fees to prevent laundering
  (*e.g.*, instead of paying full node fee for each transaction,
  a light client might announce desired transaction, and a transaction
  to move remaining funds to clean wallet).

## Discussion

An incremental path towards highly available network of full nodes
that provides a simple user experience could start with a
donation-based approach. If it's the case that the market rate of
donations is too low to incentivize a highly available network, Celo
could augument LES and reference implementations of full nodes to
include something like reputation tracking mechanisms.

## Related

Summary of Celo-based discussions and work:

* https://github.com/celo-org/celo-blockchain/pull/210
* https://github.com/celo-org/celo-blockchain/pull/212
* https://github.com/celo-org/celo-blockchain/pull/398
* https://github.com/celo-org/celo-blockchain/issues/209
* https://github.com/celo-org/celo-blockchain/issues/214
* https://github.com/celo-org/celo-blockchain/issues/216
* https://github.com/celo-org/celo-blockchain/issues/392
* https://github.com/celo-org/celo-blockchain/issues/296
* https://github.com/celo-org/celo-blockchain/issues/359

Related discussions and work in other blockchain communities:

* [Incentives for running full Ethereum nodes](https://ethresear.ch/t/incentives-for-running-full-ethereum-nodes/1239/15)
* [Economic incentive for running Ethereum nodes](https://vipnode.org/)
* [Proposal: Reward full (Cosmos) nodes](https://github.com/cosmos/cosmos-sdk/issues/1541)
