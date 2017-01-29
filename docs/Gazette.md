# Blockchain Gazettal

Periodicaly, the Notary:

 * Generate a Multi-Part Hosting Obligation Contract (HOC) describing archived records
 * Make timely public commitment to store and publish archived objects under the terms of the HOC

The HOC is a four layer Directed Acyclic Graph (DAG) comprised of JSON files, where each layer references the lower layer by cryptographic hash of it's contents, making it a "Merkel-DAG" structure.

Each document in this Merkel-DAG has a specific purpose. Some parts of the structure are always public, other parts may or may not be public depending on the application.

![ERD of Merkel-DAG](./hoc.png)

The public parts of the HOC are published in a directory-like structure with a content-address in it's path. Two parts have fixed names (`proof.json` and `proof.sig`), while all the other parts are named consistent with their content-addresses (meaning the file name is equivalent to the IPFS address of the file contents).

Content-addressing is performed consistent with the IPFS content-addresses format, for example `QmNn2peeUaJFxnPVsFjriVxnPZKkh4y2EfRqpEHQ8EYXQr`. Content addresses may refer to indifidual files or directory-like collections of files (and subdirectories, recursively).

The following examples assume the this address is the address of the directory-like structure that has been gazetted in the blockchain:

 * `/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/`

TODO:

 * create and publish better example files, and update the referece to use them.


## HOC Proof

The HOC Proof is used to validate the Hosting HOC prior to potentially intensive computation or network activity. 

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
 * TODO: determine sensible value for MAX_SIZE. 350KB? 
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

If `proof.json` and `proof.sig` both exist and neither exceed MAZ_SIZE, the next step is to ensure that proof.json validates agains the json schema. Assuming `proof.json` and `proof.schema` are both in the current working directory, the `jsonschema` program could be used like this:

```bash
jsonschema -i sample.json sample.schema
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


## HOC_HEAD

Always public JSON document of arbitrary size, containing a list.

This means the full path of the HOC_HEADER (including content-address directory path) will look something like ``/ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/QmNn2peeUaJFxnPVsFjriVxnPZKkh4y2EfRqpEHQ8EYXQr`.

Each element in the list contains:

 * HOC_DETAIL: content-address of HOC Detail document
 * DURABILITY: DATE until which the HOC Detail is promised available
 * NETWORK: Access to HOC_DETAIL is physically restricted to the private network of the business identified here. Optional field, if absent then it means the HOC_DETAIL is available from the Internet.
 * AC_PROTOCOL: Content-address of access control protocol
 * AC_CODE: Per AC_PROTOCOL, code word defining access control terms for this specific record.


## HOC_DETAIL

JSON document of arbitrary size, containing a dictionary and a list. This document may be public, private or selectively published under thetaccess control terms (AC_PROTOCOL, AC_CODE) associated with it in the HOC_HEAD.

Each element in the list contains:

 * OBJECT: content-accress of the notarised digital asset
 * DURABILITY: DATE until which the OBJECT is promised available

