# Blockchain Gazettal

Periodicaly, the Notary:

 * Generate a Multi-Part Hosting Obligation Contract (Full HOC) describing archived records
 * Make timely public commitment to store and publish archived objects under the terms of the HOC

The Full HOC is a four layer Directed Acyclic Graph (DAG) comprised of JSON files, where each layer references the lower layer by cryptographic hash of it's contents, making it a "Merkel-DAG" structure.

Each document in this Merkel-DAG has a specific purpose. Some parts of the structure are always public (first two layers, HOC Proof and HOC Header), other parts may or may not be public depending on the application (HOC Detail and Notarised Object).

![ERD of Merkel-DAG](./hoc.png)

The public parts of the HOC are published in a directory-like structure with a content-address in it's path. Two parts have fixed names (`proof.json` and `proof.sig`), while all the other parts are named consistent with their content-addresses (meaning the file name is equivalent to the IPFS address of the file contents).

Content-addressing is performed consistent with the IPFS content-addresses format, for example `QmNn2peeUaJFxnPVsFjriVxnPZKkh4y2EfRqpEHQ8EYXQr`. Content addresses may refer to indifidual files or directory-like collections of files (and subdirectories, recursively).

The following examples assume the this address is the address of the directory-like structure that has been gazetted in the blockchain:

 * `/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/`


## HOC Proof

The HOC Proof is used to validate the Full HOC prior to potentially intensive computation or network activity. 

To ensure efficient auditability of notaries, an independant observer should be able to parse the blockchain and extract OP_RETURN transactions that look like IPFS addresses. For each IPFS address-like record found, the observer should be able to ignore invalid HOCs cheaply. The purpose of the HOC Proof is to provide a compact and verifyable reference to the HOC_HEAD docuent (which may be less compact).

 * The Gazetted address MUST contain a HOC Proof, comprised of two files, `proof.json` and `proof.sig`.
 * `proof.json` MUST be valid per the `proof.schema` JSON schema specification.
 * `proof.json` MUST contain a PROTOCOL attribute, which identifies the protocol for processing the Merkel-DAG. The value of the PROTOCOL attribute MUST be `ausditigal-nry/1.0`.
 * `proof.json` MUST contain a SIG_DATE attribute containing an ISO 8601 formatted date and time string identifying when the notary notionally created the `proof.sig`
 * `proof.json` MUST contain a NOTARY attribute, which references the business identifier of the notary in URN format consistent with AusDigital DCL and DCP specifications.
 * `proof.json` MUST contain a PUB_KEY attribute, which contains a public key (TODO: what format? how encoded?)
 * The public key in the PUB_KEY attribute of `proof.json` MUST be published in the DCP entry of the NOTARY business, as a valid (not revoked) key on SIG_DATE
 * `proof.sig` MUST be a valid signature of `proof.json`, using the public key identified in `proof.json`
 * `proof.json` MUST contain a DURABILITY attribute, the value of which is an ISO 8601 formatted date and time string.
 * The DURABILITY date MUST be later than SIG_DATE by no less than one month.
 * The NOTARY MUST commit their reputation to the availability of the HOC until the DURABILITY date.
 * `proof.json` MUST contain a HOC_HEAD attribute with a value that is a valid content-address hash.

TODOs

 * TODO: notary protocol naming... probably better to have an ausditital naming standard (ausdigital-foo/version) and reference it here, saying "PROTOCOL must be valid version of ausdigital-nry per the ausdigital naming standard"
 * TODO: insert JSON reference example of HOC_PROOF.json
 * TODO: make JSON Schema document defining a valid HOC_PROOF
 * TODO: create canonical reference for this specification, including version numbers etc. Should it be a URL? it can't be a self-referencing IPFS address :) And I'm not sure that an IPNS address is the right idea.
 * TODO: Should the DCP define endpoints for accessing notarised objects? for example, if I am a notary, I would publish (possibly multiple: HTTPS URL, IPFS true/false) endpoints for accessing my notarised objects.


### Validating HOC Proof

The first step in validating the Merkel-Dag is to check that the proof.json exists and is no larger than MAX_SIZE.

TODO: determine reasonable MAX_SIZE. This will be the larger of:

 * gpg2 detached sig
 * json document
 * some nice round number of KB, 350??

For example, assuming the blockchain contains `/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/`, the validation begins with:

 * test for the existance of `/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/proof.json`
 * if the proof.json is larger than MAX_SIZE, it is not valid. There is no need to download more than (MAX_SIZE +1 byte) in this step.
 * test for the existance of `/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/proof.sig`
 * if the proof.sig is larger than MAX_SIZE, it is not valid. There is no need to download more than (MAX_SIZE +1 byte) in this step.

If `proof.json` and `proof.sig` both exist and neither exceed MAZ_SIZE, the next step is to ensure that `proof.json` validates agains the json schema. Assuming `proof.json` and `proof.schema` are both in the current working directory, the `jsonschema` program could be used like this:

```bash
jsonschema -i proof.json proof.schema
```

If the schema is valid, it will contains a `NOTARY` element containing the business identifier of the party responsible for notarisation. The value of this attribute is a business identifier which can be resolved via the AusDigital [Digital Capability Lookup](https://ausdigital-dcl.readthedocs.io), which will yield an AusDigital [Digital Capability Provider](https://ausdigital-dcp.readthedocs.io) where the public keys of that organisation can be discovered.

Note:

 * Multiple public keys may be associated with a business in the DCP register
 * The value of the PUB_KEY attribute in `proof.json` MUST be listed in the DCP entry for the NOTARY
 * The DCP listing of the PUB_KEY signature MUST NOT have a PUBLISHED date later than the SIG_DATE attribute in `proof.json`
 * If the PUB_KEY is revoked in the DCL entry for the NOTARY, it MUST NOT be revoked before the value of SIG_DATE attribute in `proof.json`

If, using NOTARY identifier and the DCL/DCP mechanism, the PUB_KEY can not be shown to be valid (non-revoked) public key of the notary on SIG_DATE, then `proof.json` is not valid.

If the public key is valid (and saved as `notary_pub.key`), you may import it to the GnuPG program like this:

```bash
gpg2 --import notary_pub.key
```

If the public key is valid, the next step is to confirm `proof.sig` is in fact a digital signature of `proof.json` that was made with the private key coresponding to the PUB_KEY attribute of `proof.json`.

Assuming you have imported the verified valid key into the GnuPG program, and have `proof.json` and `proof.sig` in your present working directory, you can confirm the signature like this:

```bash
gpg2 --verify proof.sig proof.json
```

After verifying proof.json with proof.sig, it is safe to process the HOC Header


## HOC Header

The HOC Header is processed after the HOC Proof has been validated. It is essentially an arbitrarially long list of references to HOC Detail records. It also contains metadata about access and availability of HOC Detail records.

 * The Full HOC (directory-like content-address, gazetted to the blockchain) MUST contain a file with a valid content-address, that is referenced by the HOC_HEAD attribute of `proof.json` (the "HOC_HEAD" file)
 * The contents of the HOC_HEAD file MUST be json that is valid per `hoc_head.schema` JSON schema
 * HOC_HEAD MUST contain a list of one or more elements.
 * Each element in HOC_HEAD list MUST contain a HOC_DETAIL attribute, which is a valid content-address of a file.
 * Each element in HOC_HEAD list MUST contain a DURABILITY attribute.
 * The value of DURABILITY attribute MUST be an ISO 8601 formatted string describing the DATE until which the HOC Detail is promised available.
 * Each element in HOC_HEAD list MUST contain a NETWORK attribute.
 * The value of NETWORK attribute MUST be a valid business identifier URN per the DCP and DCL specifications.
 * Each element in HOC_HEAD list MUST contain a AC_CODE attribute.
 * The value of AC_CODE MAY be an empty string.
 * The HOC Header MAY be published directly with it's content-address (e.g. `/ipfs/QmNn2peeUaJFxnPVsFjriVxnPZKkh4y2EfRqpEHQ8EYXQr`).
 * The HOC Header MUST be published with it's content-address as it's name within the directory-like collection that was gazetted to the blockchain (e.g. `/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/QmNn2peeUaJFxnPVsFjriVxnPZKkh4y2EfRqpEHQ8EYXQr`).

The AC_CODE partially defines the protocol for accessing the record referenced in the HOC_DETAIL attribute. The meaning of the AC_CODE is dependent on the NETWORK. In other words, the AC_CODE is interpreted in the context of the NETWORK.

Because the network is a business identifier URN, the AC_CODE interpretation context can be determined by DCP query...

 * TODO: document DCP document and process types for interpreting AC_CODE

If the NETWORK is the URN of the notary, the DCP lookup of RECORD_ACCESS_PROTOCOL MUST return `ausdigital-nry/1` or higher version. Otherwise they can be anything (including undefined).

 * TODO: cross reference ausdigital specification naming specification

If the DCP document type / process type of the HOC_HEADER list item's NETWORK is `ausdigital-nry/1`, then the allowable value of AC_CODE are:

| Access Protocol | AC_CODE | HOC Detail | Notarised Object | Comment                         |
|-----------------|---------|------------|------------------|---------------------------------|
| ausdigital-nry/1 | 0      | Public     | Public           | Transparently auditable open data (e.g. public HOC documents). |
| ausdigital-nry/1 | 1      | Public     | Private          | Independently auditable secret data (e.g. business transaciton records, such as eInvoices). |
| ausdigital-nry/1 | 2      | Private    | Public           | Plausibly deniabile open data (that can be subsequently rendered transparently auditable by publishing the HOC Detail). |
| ausdigital-nry/1 | 3      | Private    | Private          | Privately auditable secret data (e.g. Corporate records). |
| [other] | [any] | [either] | [either] | If Access Protocol is not `ausdital-nry/1` (or later), arbitrary (localy meaningful) AC_CODE may be used. |


### Validating HOC Header

 * TODO: document DCP lookup procedure for determining AC_CODE interpretation

note: this can be cached: when processing a HOC Header, the value of DCP lookups MAY be assumed not to change between HOC_HEADER list items.

For each HOC_HEADER list item:

 * If the NETWORK is not equal to the `proof.json` NOTARY, and if DCP lookup of RECORD ACCESS PROTOCOL for the NETWORK does not resolve to `ausdigital-nry/1`, then this specification does not describe how to validate that HOC_DETAIL record. Reffer to the appropriate protocol specification for interpretation of AC_CODE values in that context. Do not proceed with validating the listed HOC Header or any HOC Details it seems to reffer to (even if the syntax seems identicacompatabledwith tal-nry/1` protocol).

If the NETWORK is equal to the `proof.json` NOTARY, or if the DCL lookup of RECORD ACCESS PROTOCOL for the NETWORK resolves to `ausdigital-nry/1`, then apply the following validation rules:


If AC_CODE is "0" or "1":

 * If the HOC_HEADER item DURABILITY is a future date, the content-address MUST be directly available (for example `/ipfs/QmNn2peeUaJFxnPVsFjriVxnPZKkh4y2EfRqpEHQ8EYXQr`
 * If the HOC_HEADER item DURABILITY is a future date, and the `proof.json` DURABILITY is also a future date, the content address MUST be available within the directory-like container of the Full HOC (for example `/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/QmNn2peeUaJFxnPVsFjriVxnPZKkh4y2EfRqpEHQ8EYXQr`)


If AC_CODE is "2" or "3":

 * If the HOC_HEADER item DURABILITY is a future date, and the API Token has valid identity claim, the content-address MAY be avilable via the notary API (`GET /private/{content_address}`).
 * If the HOC_HEADER item DURABILITY is a future date, and the API Token has valid identity claim, and the API Token identity claim matches the `proof.json` NOTARY, the content-address MUST be avilable via the notary API (`GET /private/{content_address}/`).
 to the business identity of the NETWORK, the content-address MUST be avilable via the notary API (`GET /private/{content_address}`).



When notarised objects are submitted to the notary API with AC_CODE "1", "2" or "3" (e.g. `POST {object+params} /private/`), the RESTRICT_LIST parameter contains a list of business identifiers permitted to access the object. This RESTRICT_LIST identifies the business identifiers of all parties with rights to access the record.

When notarised objects are submitted to the notary API with AC_CODE "1", the RESTRICT_LIST attribute only applys to the notarised object. When notarised objects are submitted to the notary API with AC_CODE "2" or "3", the RESTRICT_LIST parameter MUST be applied to both the HOC_DETAIL and the notarised object. This means:

 * If the HOC_HEADER item DURABILITY is a future date, and the API Token has valid identity claim, and the API Token identity claim matches one of the identities in the records RESTRICT_LIST, and the AC_CODE is "3", then the HOC_DETAIL content-address MUST be available via the notary API (`GET /private/{content_address}/`)
 * If the HOC_HEADER item DURABILITY is a future date, and the API Token has valid identity claim, and the AC_CODE is "1", "2" or "3"; then the HOC_DETAIL content-address MUST be available via the notary API (`GET /public/{content_address}/`)
 * TODO: cross reference API spec (request notarisation on the API)
 * TODO: ensure API spec includes AC_CODE parameter (validate if /private/ AC_CODE in (2,3), if /public/ AC_CODE in (0,1), if AC_CODE==0 RESTRICT_LIST empty (no restrictions), if AC_CODE!=0 RESTRICT_LIST must not be empty).



### Notary Reputation

Once a `proof.json` has been validated, the availability (or otherwise) of the HOC Header, within the timeframe of `proof.json` DURABILITY, this MAY be considered to impact the objective reputation of the Notary.

The availability or otherwise of HOC Headers may or may not be directly related to the objective global reputation of the Notary.

 * If the HOC_HEAD.NETWORK is a URN of the same business as the `proof.json` NOTARY, and if the RECORD_ACCESS_PROTOCOL of that business (DCP lookup) is not `ausdigital-nry/1` (or later version), then the notary has absconded their responsability. This MAY be interpreted as strongly negative impact on the reputation of the notary.
 * If the HOC_HEAD.NETWORK is a URN of a different business as the `proof.json` NOTARY, the availability or otherwise of HOC_DETAIL (and notarised objects) MUST NOT be interpreted as impacting the reputation of either business.
 * The DURABILITY date/time represents the limit of promised availability of the notarised object.
 * If the HOC_DETAIL NETWORK is the same business as NOTARY, that means the Notary is taking responsability for the promise of availability. Failture to meet terms of AC_CODE and DURABILITY will detrimentally impact the reputation of the Notary.
 
If the HOD_HEAD.NETWORK identifies a business other than the `proof.json` NOTARY, the DURABILITY and AC_CODE represent claims (by the notary) about promises made by another business. The reason these MUST NOT be interpreted as impacting objective reputation of either party is that it is impossible to distinguish false claims from true claims and unkept promises.

In addition to global objective notary reputation (which can be based on transparently auditable open data), organisations with priveleged access to the notary API (through RESTRICTION_LIST priveleges to AC_CODE "1", "2" and "3" notarised objects and AC_CODE "2" and "3" HOC_DETAIL records) may produce private objective reputation scores.


## HOC Detail

The HOC Detail is a JSON document of arbitrary size, referenced by it's content-address in the HOC Header list.

 * The HOC Detail MUST contain a list of one or more elements.
 * The HOC Detail MUST be validate against the `hoc_detail.schema` JSON schema.


Every element in the HOC Detail list:

 * MUST contain a DURABILITY attribute, which contains an ISO 8601 formatted date string.
 * The DURABILITY date MUST NOT be less than one month ahead of the `proof.json` SIG_DATE attribute.
 * The DURABILITY date MUST NOT be less than the coresponding HOC_HEADER durability date.
 * MUST contain an OBJECT attrubute, which contains a content-address of some notarised object. 

The elements in the HOC Detail inherits an AC_CODE and NETWORK from the reference to the HOC Detail in the HOC Header.

 * If the inherited AC_CODE is "0", and `proof.json` NOTARY identifies the same business as the inherited NETWORK, and the listed DURABILITY date is in the future, then the OBJECT this HOC Detail must be available through the API (e.g. `GET /public/{content_address}`)
 * If the inherited AC_CODE is "1", "2" or "3"; and `proof.json` NOTARY identifies the same business as the inherited NETWORK, and the listed DURABILITY date is in the future, and the API Token has a valid identity claim, and the identity in the API token identifies a business in the RESTRICT_LIST of the object, then the OBJECT this HOC Detail MUST be available through the API (e.g. `GET /private/{content_address}`)


TODO:

 * create `hoc_detail.schema`