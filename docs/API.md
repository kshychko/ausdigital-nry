# Notary API

The notary API can be used for two purposes:

 * POST methods that task the notary with notarising a digital record (this may or may not involve storing the object in the notary's archive).
 * GET methods that access notarised data from the notary's archive (this may or may not require authentication, and may or may not be subject to content-based access control rules)

Perhaps unsurprisingly, the notary API does not generally furnish any PUT, PATCH or DELETE methods. Notary records are imutable until disposal, which is scheduled at the time of notarisation.

Notary records are addative; the only way to change the parameters of notarisation is to make a subsequent notarisation request with different parameters (the notary will meet obligations of both requests).

The notary API is comprised of two REST/JSON resources, `/public/` and `/private/`. These work in similar ways, but have different access control rules. Separate endpoints are used (rather than a parameter) to minimise risk of accidentialy publishing private objects ("poka-yolk" principle).

Implementations  MAY be allow users to restrict client access to one or the other endpoint on a per-API token basis.


| Verb   | /public/*               | /private/*              |
|--------|-------------------------|-------------------------|
| GET    | no token required       | token + ACL             |
| POST   | token + ACL             | token + ACL             |
| PUT    | generally not available | generally not available |
| PATCH  | generally not available | generally not available |
| DELETE | generally not available | generally not available |


The Notary API is a REST/JSON protocol, layered on HTTP. Implementations:

 * MUST use HTTPS (RFC2818).
 * MUST use Content-Type: multipart/form-data (RFC2388)
 * MUST NOT use Content-Type: application/x-www-form-urlencode (RFC1876 is NOT supported)
 * MAY explicitly declare Content-Transfer-Encoding: base64
 * MUST NOT rely on notary semantic information in HTTP headers


## POST object to Notary

The POST verb is used to request notarisation. The posted body contains two parts, `object` and `parameters`.

Posting an object to the `/public/` or `/private/` notary resource causes the notary business to incur cost and obligation. The notary may refuse, or may agree under commercial terms. For this reason, POST requests always must be authenticated with a JWT header issued under the `ausdigital-idp/1` specification.

Assuming the current working directory contains the object (as `object.foo`) and parameters (as `param.json`), the following curl command will post the message to the notary endpoint:

```curl -X POST \
 -H "Content-Type: multipart/form-data" \
 -H "Authorization: Bearer <API_TOKEN>"
 -F "object=@object.foo" \
 -F "param.json" \
 <NRY_URL>
```

(Replace <NRY_URL> with HTTPS URL discovered from the Digital Capability Publisher of the business providing the Notary Service, and replace <API_TOKEN> with the JWT issued by the `ausdigital-idp/1` Identity Provider).

The protocol places no physical restriction on the `object.foo` POSTed to the notary API, however there may be some practical limits from the underlying HTTP infrustructure. For example, there may be some limit to the maximum file size than can be reliably POSTed.

The `param.json` file:

 * MUST be a JSON file that is valid per `nry_post_param.schema` JSON schema.
 * MUST contain a DURABILITY attribute that is an ISO 8601 formatted date and time string
 * The DURABILITY date must be not less than one month in the future
 * MUST contain a NETWORK attribute that is a valid business identifier URN per `ausdigital-dcp/1` specification.
 * MUST contain an AC_CODE attribute
 * MUST contain an RESTRICT_LIST attribute, that is a JSON list of zero or more elements that are valid business identifier URNs per `ausdigital-dcp/1`.
 * If the NETWORK value is a business identifier of the notary itself, then the AC_CODE MUST be interpreted per `ausdigital-nry/1`. 
 * If the NETWORK value is not a business identifier of the notary, the AC_CODE MAY be interpreted as AC_CODE = 3.
 * If the NETWORK value is not a business identifier of the notary, the notary MAY silently fail to store the notarised object.
 * If the NETWORK value is not a business identifier of the notary, the Notary API resource must be `/private/` (alternate NETWORKs MUST NOT be used with the `/public/` resource). 
 * If the Notary API resource is `/public/`, the AC_CODE must be "0".
 * If the AC_CODE is "1", "2" or "3" (or may be interpreted as "3"), the Notary API resource must be '/private/'.


When the notary recieves a valid notarisation request, if it does not refuse the request and does not experience technical difficulties, then it MUST:

 * place notarised object in a system of record (unless the `param.json` NETWORK value is not a business identifier of the notary, in which case the notary MAY place the notarised object in a system of record)
 * reference API Spec (return HTTP 200 response code, headers, etc)
 * return HTTP body that is JSON document that is valid per `nry_post_response.schema` JSON schema.
 * The body must contain a DOC_ID attribute containing a verifyable content-address of the notarised object, using a hash and encoding scheme that is a valid IPFS address.

TODO:

 * re-align response objects with https://app.swaggerhub.com/domains/ausdigital/ErrorModel/1.0


When the notary recieves an invalid notarisation request, or if it recieves a valid request that it choses to refuse, or if it experiences technical difficulties, then it:

 * MUST return an appropriate HTTP response code (see swagger specification)
 * The body MUST NOT contain a DOC_ID attribute
 * etc... TODO

The DOC_ID returned in the body of successful POSTs (HTTP code 200 responses) is a valid content identifier. This DOC_ID is subsequently used as notarised object identifier in blockchain Gazettal. It is also the DOC_ID used in `GET /public/{DOC_ID}/` and `GET /private/{DOC_ID}/` API calls.
  

`POST {object+params} /public/`

`POST {object+params} /private/`

parameters:
 * DISPOSAL_DATE
 * RESTRICT_LIST
 * ALT_NETWORK

return:
 * 200 + address-hash of POSTed object
 * TODO: what other meta data?
 * TODO: enumarate error conditions...



## Search Notary Archives

Given the appropriate authorisation and other circumstances, can be used as an identifier to access the object in the system of record

TODO:

 * refactor and elabrate content in Gazzette.md
