**[Back to AusDigital.org](http://ausdigital.org/)**

# ADBC Notary

 * ![raw](http://rfc.unprotocols.org/spec:2/COSS/raw.svg)
 * Editor: Chris Gough
 * Contributors:

This document describes the Notary (NRY) specification, which provides an irrefutable
history of a specific "contract" (e.g. an invoice and it's status lifecycle), pegged to
a blockchain distributed ledger. This service supports dispute resolution and provides
the foundation platform for financial services such as debtor financing. Notarisation
also makes it possible for the business system to audit the reliability of any TAP
Gateway that it uses.


## Goals

Facilitate features in the TAP protocol that:

 * Prevent post-hoc "Fraud of Commission" (retrospective creation of false document history).
 * Prevent post-hoc "Fraud of Ommission" (surrupticious destruction of legitimate document history).


Balance information security with utility:

 * Sufficient public data for transaction stakeholders to independently verify integrity of shared-secret documents.
 * Information from private documents is not leaked into the public domain, even in a future where tools exist to overcome contemporary cryptographic methods.
 * Ensure notarised object sources are identified (to a known and meaningfull level of identity assurance), but do not leak information about subjects that could be used to anlayse communication or transaction traffic.
 * Publically auditable (transparent), to enalble notary reputation to be based on independent performance evaluation using transparent, non-repudiable and objective data. In adition to the relevent data being open, the cost of monitoring and verifying the activities of a notary must not be prohibitive.


Sustainable and secure infrastructure:

 * Socially responsible, blockchain-efficient public storage of existance proofs. This means low externalised community cost (impact on the shared ledger) even if a very large number of objects are notarised in a given time frame.
 * Leverage the strongest consensus product available (maximise resistance to Sybl attacks).
 * Leverage the most efficient proof market available (minimise cost of proof at the given consensus-strength).
 * Avoid single point of failure in the network that stores and distributes existance proof.
 * 


These are achieved by:

 * Notarising all objects (in a timeframe) into a proscribed Merkel-DAG data structure that is pegged to the Blockchain with a single record, which references a minimal signed proof document that can be verified before accessing a significant data volumes.
 * Distributing the pegged Merkel-DAG proof structure with a decentralised, content-addressable memory system (modelled on and compatible with the Inter Planetary File System, IPFS).
 * Avoid propietary blockchains. Adopt the largest public blockchain with the highest market capitalisation (BitCoin), thereby sourcing work-proof from an open market of commodity mining services and multiple open source -oftwaresimplementations.



## Status

This spec is an early draft for consuiltation.

This specification aims to support the Australian Digital Business Council
[eInvoicing initiative](https://ausdigital.org), and is under active
development at
[https://github.com/ausdigital/ausdigital-nry](https://github.com/ausdigital/notary).

Comments and feedback are encouraged and welcome. Pull requests with improvements are welome too.

## Links

This spec depends on, and should be read in conjunction with the AusDigital [Identity Provider (IDP)](https://ausdigital-idp.readthedocs.io) specification.

The AusDigital [Transaction Access Point (TAP)](https://ausdigital-tap.readthedocs.io) specification depends on this document. Note, while this specification describes a generic notary interface, the TAP specification provides further restriction on the use of the notary.
 

## Licence

Copyright (c) 2016 the Editor and Contributors. All rights reserved.

This Specification is free software; you can redistribute it and/or modify it under the
terms of the GNU General Public License as published by the Free Software Foundation; 
either version 3 of the License, or (at your option) any later version.

This Specification is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program;
if not, see [http://www.gnu.org/licenses](http://www.gnu.org/licenses).


## Change Process

This document is governed by the [2/COSS](http://rfc.unprotocols.org/spec:2/COSS/) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in
RFC 2119.
