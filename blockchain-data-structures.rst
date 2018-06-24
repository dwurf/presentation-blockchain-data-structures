:title: Blockchain data structures
:data-transition-duration: 300
:css: css/presentation.css

----

Blockchain Data Structures
==========================

.. image:: blockchain-3212304_1920.jpg
    :height: 341px
    :width: 800px

Darren Wurf

----

Introduction
========================================================================

<h1>BLAH</h1>

Introductory material should set the expectations of this talk, i.e. we will be talking about blockchains from a software engineering perspective, to demystify them and better understand what they actually do.

Bitcoin will be used for the examples unless noted otherwise.

----

Blockchain
========================================================================

The **blockchain** is a **shared ledger** where each new **block of transactions** is signed with a **Nakamoto signature**.

----

Shared ledger
========================================================================

.. image:: timestamp-server.svg

* Allows **shared-write** to a **distributed ledger** amongst **mutually distrusting parties**

1. Executive view: A blockchain is a shared decentralized ledger, enabling 
business disintermediation and trustless interactions, thereby lowering 
transaction costs

The shared ledger can be seen as a series of blocks, each of which is a single document presented for signature. Each block consists of a set of transactions built on the previous set. Each succeeding block changes the state of the accounts by moving money around; so given any particular state we can create the next block by filling it with transactions that do those money moves, and signing it with a Nakamoto signature. 

1. A blockchain is a peer-to-peer protocol for trust-
less  execution  and  recording  of  transactions  se-
cured by asymmetric cryptography in a consistent
and immutable chain of blocks – the blockchain
developers and technology view.

2. A blockchain is a shared append-only distributed
database with full replication and a cryptographic
transaction permissioning model – the IT architect
and data management view.


----

Nakamoto signature
========================================================================

A Nakamoto signature is a device to allow a group to agree on a shared document. To eliminate the potential for inconsistencies (disagreement), the group engages in a lottery to pick one person's version as the one true document. That lottery is effected by all members of the group racing to create the longest hash over their copy of the document. The longest hash wins the prize and also becomes a verifiable 'token' of the one true document for members of the group: the Nakamoto signature. 

----

Key points to get across
========================================================================

 * Blockchains provide a ledger with **shared-write** capability to a bunch of entities who **do not trust each other**

Blockchains enable cryptocurrency
----------------------------------------------------

Cryptocurrency Bitcoin had a very specific set of requirements that was not solved by any technology available at the time. Specifically, there was no way to prevent double-spending of digital currency (including traditional currencies) without a trusted intermediary maintaining and monitoring a transaction ledger.

Previous attempts at designing a digital currency had not solved this problem, but one design **b-money** proposed a solution in the form of a global timestamp server, "kind of like usenet". 

Bitcoin solved this problem by introducing the concept of a global, peer-to-peer timestamp network where anyone could record transactions, but the network itself would not allowing double-spending.

----

Compared to traditional databases
====================================================

Properties of blockchains:

 * Log-structured, immutable, append-only, peer-to-peer
 * Support for cryptographic controls down to the individual record level
 * ACID compliant? **No**: see `SALT <http://www.ise.tu-berlin.de/fileadmin/fg308/publications/2017/2017-tai-eberhardt-klems-SALT.pdf>`_
 * CAP theorem: AP - eventually consistent - miners vote using Proof of Work

----

Compared to traditional databases
====================================================

Downsides of blockchains

 * Very high cost per transaction (power consumption / specialised compute)
 * Very high latency for transaction confirmation (e.g. 6 blocks / 1hr)
 * Low capacity and throughput
   * Bitcoin is limited to 1MB every 10 minutes, averaging about 7 transactions per second
 * Dependent on expensive consensus tools, e.g. through Proof of Work (mining)
 * Requires incentives to sustain the network, e.g. block reward

----

Blockchains are a **distributed ledger**
----------------------------------------------------

Evolution of the ledger concept. Not sure how important this is for an engineering discussion. Are they a ledger or do they just allow for a ledger to be built on top of them? Without the ledger concept how would you pay miners to run and secure the network?

----

Questions arising from the above
--------------------------------

 * What are the properties of these data structure?
 * How do they compare to other data structures?
 * What is the key innovation that blockchain brings to the table?
     * From a data structure perspective
     * From a software perspective (Don't focus on this too much, the purpose is to explain why the data structure is new)

----

The data structures
===================

 * Chain
 * Blocks
   * Hashes and nonce
   * Transactions (or whatever goes in the block for this blockchain)
   * Reward
 * Merkle trees
 * Mempool
 * Structures used for peer to peer communication

----

The chain
=========

.. image:: proof-of-work.svg

* Each block contains the hash of the previous block
* Blocks contain a header and some transaction data. 
* The first transaction is called the coinbase and is allowed to create new bitcoin.

----

The chain
=========

.. image:: proof-of-work.svg

* Miners increment the nonce to change the hash of the current block
* A block is published once a hash is found that meets the difficulty threshold
* For example, if the difficulty is ``0x00001b...`` the miners must find a hash with lower starting bits


----

The block
================

+---------+---------------+----------------+-------+-------+--------+
| version | hashPrevBlock | hashMerkleRoot | nTime | nBits | nNonce |
+---------+---------------+----------------+-------+-------+--------+
| tx0 (coinbase)          | tx1..n                                  |   
+---------+---------------+----------------+-------+-------+--------+

.. code:: c++

    // Bitcoin block header
    int32_t nVersion;        // Block version
    char[32] hashPrevBlock;  // sha256
    char[32] hashMerkleRoot; // sha256
    uint32_t nTime;          // Unix timestamp
    uint32_t nBits;          // Difficulty target
    uint32_t nNonce;         // Increment nonce to "mine" (change the hash)

.. note::

    Merkle trees: A kind of "cryptographic summary" of the data in the block.
                  They allow us to hash only the headers.

----

The Mempool
================

* To publish data on the blockchain, people sign a document (e.g. transaction) and broadcast it to nodes on the network
* The published document must satisfy the network rules (e.g. no double-spend)
* There is often a fee to publish, paid to the miners
* Nodes store valid, `unconfirmed transactions<https://blockchain.info/unconfirmed-transactions>`_ in the **mempool**

----

The Transaction
================

.. image:: transactions.svg

* A transaction is a signature of the inputs and outputs of existing coins
* Inputs are ?previous transactions?
* Outputs are ?locking script?
* TODO: look at the source code

----

The Merkle Tree
================

+---------+---------------+--------------------+-------+-------+--------+
| version | hashPrevBlock | **hashMerkleRoot** | nTime | nBits | nNonce |
+---------+---------------+--------------------+-------+-------+--------+

* The merkle tree summarises the data (transactions) stored in the block
* The root of the tree is stored in the block header
* Only the header of the block is hashed by miners, individual transactions are not

----

The Merkle Tree
================
.. image:: Hash_Tree.svg
    :height: 509px
    :width: 800px

* Leaf nodes are the hash of the data blocks
* The intermediate nodes are the hash of their children
* The root is stored in the block header

----

Compacting old blocks
================================

.. image:: reclaiming-disk-space.svg
    :height: 509px
    :width: 800px

* Historic transactions can be pruned to save space
* Nodes can store just the parent node for branches they aren't interested in
* Blocks and transactions can still be validated using the parent nodes

----

Simplified Payment Verification
================================

.. image:: simplified-payment-verification.svg
    :height: 315px
    :width: 800px

* Full nodes store the entire blockchain history
* SPV allows users to use the blockchain without storing the full history
* A mobile wallet can download just the block headers and relevant branches

----

The Mempool
================================

----

:data-x: r0
:data-y: r1000

What can we actually do with blockchains?
========================================================================

* Record transactions (obviously)
* A distributed computer - all nodes compute the exact same value, which enables:
* Smart contracts: the ability to 
* Proof-of-possession for some data at some time e.g. https://canyouproveit.io/
* Creation of automatic markets, particularly for high-volume, low-value transactions

----

:data-rotate: 90

Code
====

From `src/primitives/block.h`:

Nodes collect new transactions into a block, hash them into a hash tree,
and scan through nonce values to make the block's hash satisfy proof-of-work
requirements.  When they solve the proof-of-work, they broadcast the block
to everyone and the block is added to the block chain.  The first transaction
in the block is a special one that creates a new coin owned by the creator
of the block.

Links
========

* Whitepaper: https://nakamotoinstitute.org/bitcoin/ (read the references too!)
* Explanantion: https://www.vpnmentor.com/blog/ultimate-guide-bitcoin/
* Protocol structures: https://en.bitcoin.it/wiki/Protocol_documentation#Common_structures
* Properties of blockchains: SALT: http://www.ise.tu-berlin.de/fileadmin/fg308/publications/2017/2017-tai-eberhardt-klems-SALT.pdf
* Real-time transaction view: https://blockchain.info/unconfirmed-transactions
* Real-time transaction visualisation: https://bitbonkers.com/

Glossary
========


 * Byzantine fault tolerance - resistance to failures in a distributed system
 * Merkle trees
