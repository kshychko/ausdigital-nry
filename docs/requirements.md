# Requirements

Auditor:

 * Someone who subscribes to the blockchain and evaluates the notarisation information present

Notary:

 * A business who publishes existance-proof to a public blockchain

Transaction Stakeholder:

 * Someone who is a party to, or interested in a notarised transaction
 * may or may not be an auditor
 * may resport to checking existance proof as part of verifying information related to the transaction

```
As a Transaction Stakeholder
I need the ability to verify messages have been notarised
So that I can be certain they won't be denied in the future
```

Assumptions:

 * transaction stakeholder will have access to notarised artefacts
 * transaction stakeholder may or may not have keys to decrypt notarised artefacts


```
As a Notary
I need the ability to publish proof of existance
So that I am not burdened with excessive trust
```

Assumptions:

 * cryptographic hashes prove existance sufficiently
 * the contents of notarised artefacts is not  may be, even when existance-proof is public
 * a blockchain ledger is the most appropriate way to publish existance proofs

```
As an Auditor
I need to be able to subscribe and validate all published existance proofs
So that I can analyse and follow up on transactions of interest
```

Assuptions:

 * blockchain ledger is sufficient publish/subscribe mechanism for auditors
 * realtime performance is not required by auditors (days or hours, not seconds)
 * auditors will be capable of "winnowing" irrellevent or invalid data from a message stream


## Choice of Blockchain

Proposal: Use the Bitcoin Blockchain

Justification/assumptions:

 * all private blockchains are inappropriate (transparency is required)
 * Bitcoin has the most efficient work-proof market
 * Bitcoin is the most Sybil-resistant consensus product


## Blockchain Efficiency

The volume of notarised messages could be very high (billions/year). It would be socially irresponsible, cost-prohibitive and grosely inefficient to notarise them all individually to the blockchain. Fortunately, this is entirely unessiscary. Notarised messages can be organised into a "Merkel-DAG" (Directed Acyclic Graph or "tree" of hashes of hashes), such that such that an arbitrary number of messages can be existance-proofed with a single hash.


## Content Addressable Memory

Unlike HTTP URL addresses, which identify a resource based on it's location, a "content addressible memory system" provides access to a resource based on it's content. In these systems, an address contains a hash of the content.

Proposal: Use IPFS for publishing the Merkel-DAG of existance proofs

Justification:

 * IPFS leverages existing internet infrastructure
 * A fully quallified IPFS addresses is currently 42 Bytes. Given the current stable bitcoin-core allows 83 bytes for OP_RETURN scripts, this leaves up to 9 bytes for tagging even after doubling the IPFS hash size to 64 bytes (i.e. leaving room for expansion when contemporary hash algorithms are retired)

Example:
```
NRY /ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG
```

This example uses:

 * 4 bytes for tagging `NRY `
 * 6 bytes protocol specification `/ipfs/`
 * 1 byte to identify SHA-256 hashing algorithm `Q`
 * 1 byte to identify 32 bit hash length `m` 

Note the hash itself is BASE58 encoded, so it's really only 32 bytes (even though it appears to be 44 in the above example).


## Spamm Resistance

There are circumstances when someone might write the IPFS address of a large file to the public blockchain. If an auditor were following the blockchain for all public notarisation information, they would need to download all IPFS addresses notarised to the blockchain before evaluating them. The cost of downloading very large, irrelevant files would not be insignificant.

Proposal: limit maximum size of notarisation file to 150KB

This would solve the problem because if someone writes to the blockchain the IPFS address of a file larger than this limit, an auditor could safely abort downloads that exceed this limit, knowing that they are not missing a valid notarisation information.

If the published file has limited size, but the volume of notarised messages for the coresponding time period is arbitrarially large, then the notarised file must contain pointers (also IPFS addresses) to a tree (DAG) of other files. The auditor could follow those links after validating the notarised "root".


## Spoof Resistance 

Note, proof of existance is not proof of correctness. Anyone could publish a spurious, vexatious or accidentially misleading assertion to the blockchain.

Assumptions:

 * A Notary is a business
 * A Notary would published their key's in the SMP/DCP.
 
Proposal: Notary signs notarised document with their well known public key.

Notarisation information must be signed using the same digital signature mechanism as other business documents.
