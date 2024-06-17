# did:self method specification
## Author
* [Nikos Fotiou](https://www.fotiou.gr), [MMLab/AUEB](https://mm.aueb.gr), [ExcID](https://www.excid.io)

## About
did:self is a DID method that enables DID document management without registries. 
A did:self identifier is the thumbprint of a JWK as defined in [RFC 7638](https://www.rfc-editor.org/rfc/rfc7638).
The corresponding  DID document is protected by a JSON Web Signature generated
by the private key that corresponds to the did:self identifier. The payload of this signature
is the DID document itself.

A Python3 [implementation](https://github.com/excid-io/did-self-py)

### Research
* The [SECOND](https://mm.aueb.gr/projects/second) project that uses did:self.
* The [SCN4ND](https://mm.aueb.gr/scn4ndn/) project that uses did:self.
* Application of did:self in IPFS [N. Fotiou, V.A. Siris, G.C. Polyzos,
"Enabling self-verifiable mutable content items in IPFS using Decentralized 
Identifiers", in DI2F: Decentralising the Internet with IPFS and Filecoin, IFIP Networking 2021 workshop](https://arxiv.org/abs/2105.08395)
* Application of did:self in securing routing in Named Data Networking
[N.Fotiou, Y. Thomas, V.A. Siris, G. Xylomenos and G.C. Polyzos, "Securing Named Data Networking routing using Decentralized Identifiers," in Proc. SARNET-21 workshop, IEEE International Conference on High Performance Switching and Routing (HPSR), Paris, France, June 2021](https://mm.aueb.gr/publications/12279f1a-8166-4560-aead-56dfe90df93f.pdf)
* Application of did:self in securing IoT group communication
[N. Fotiou, V.A. Siris, G. Xylomenos and G.C. Polyzos, "IoT Group Membership Management Using Decentralized Identifiers and Verifiable Credentials", in Future Internet, MDPI, vol. 4, iss. 6, 2022](https://www.mdpi.com/1999-5903/14/6/173)
* The [SNDS](https://mmlab-aueb.github.io/snds-site/) project that uses did:self

## The did:self method 
The name of this DID method is: `self`

The method specific identifier is represented as the thumbprint of a JWK, 
as defined in [RFC 7638](https://www.rfc-editor.org/rfc/rfc7638), e.g.,

```
did:self:iQ9PsBKOH1nLT9FyhsUGvXyKoW00yqm_-_rVa3W7Cl0
```

The DID document is a JSON-encoded file that must include at least
the `id` property.  

The integrity of a DID document is verified using a 
[JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515).

The header of the proof must include the `jwk` field, which is used as follows:

* `jwk` The JWK that can be used for verifying the proof. The thumbprint of this key **must** match the did:self identifier 

The payload of the proof is the DID document.


## DID document validation
Given a did:self DID, a DID document and the corresponding JWS any entity can attest whether
 or not the DID document is a 
valid document for the given DID using the following protocol.

1. Verify that the thumbprint of the `jwk` field of the header of `proof` is equal to the DID.
1. Verify the signature of the `proof` using the `jwk` field of the header.

## DID document resolution
The DID document resolution process of did:self DID is application specific. Given that
DID documents in did:self are self-certified there is no need for a secure registry.
DID documents can be included in message exchanges, they can be stored in file
system, or they can be provided as DNS records. 

## CRUD Operation Definitions
CRUD operations should implemented by the user. Our Python3 [implementation](https://github.com/excid-io/did-self-py)
can be used for this purpose. 

### Create
The Create operation initializes a did:self DID and creates a DID document. 
The JWS of the DID document is signed using the using the 
private key that corresponds to the did:self DID.

The following is a valid DID document

```JSON
{
  "id": "did:self:t_Mi-DsXvoOwL8igf7mlu2VSAJIKqhKhWIZTrYAG8XI",
  "authentication": [
    {
      "id": "#key1",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "kty": "EC",
        "crv": "P-256",
        "x": "YOGmYaMKzwTFytWHN2hGC-2VpPqGqj_sDSckB2IvCgI",
        "y": "k7iWuiXQlLXvROjdMA2WNHhGz0jxu6u41n83YupNteo"
      }
    }
  ]
}
```

The following is the corresponding proof in  Compact Serialization format with detached payload.

```
eyJhbGciOiAiRVMyNTYiLCAiandrIjogeyJrdHkiOiAiRUMiLCAiY3J2IjogIlAtMjU2IiwgIngiOiAiWU9HbVlhTUt6d1RGeXRXSE4yaEdDLTJWcFBxR3FqX3NEU2NrQjJJdkNnSSIsICJ5IjogIms3aVd1aVhRbExYdlJPamRNQTJXTkhoR3owanh1NnU0MW44M1l1cE50ZW8ifX0..T1WGz3wTmWLZer0_40hkGrZ6Qky_17dbR3y0G_kIsNvuLXlDhAdUoGnMitOGBYMN4J0vy91TPyqx0ENKTRY5aQ
```

The following is the decoded JWS header

```JSON
{
  "alg": "ES256",
  "jwk": {
    "kty": "EC",
    "crv": "P-256",
    "x": "YOGmYaMKzwTFytWHN2hGC-2VpPqGqj_sDSckB2IvCgI",
    "y": "k7iWuiXQlLXvROjdMA2WNHhGz0jxu6u41n83YupNteo"
  }
}
```


The DID document may include an arbitrary number of verification methods and relationships.
The public key(s) used as verification methods do not have to be the same as the
DID identifier, although using the same key is allowed. 

### Read
The Read operation should output the DID document and JWS. The DID document should be then validated using the process
described in [DID document validation](#did-document-validation)


### Update
With the update operation, the DID document is replaced
with a new one. A new JWS is also generated. 

### Deactivate
Since there is no registry the deactivate method is not supported. 

## Security and Privacy Considerations

### Security
This method considers two types of cryptographic keys: the key that corresponds to the 
identifier, which is used for generating the DID document proofs, and the keys used
as verification methods. Although the same key can be used in both cases,
it is recommended to use different keys so that verification keys can be rotated.

 

This method  does not provide an 1:1 relationship between a did:self DID identifier and
the corresponding DID document: there can be multiple valid DID documents for the same
identifier. This is a design choice that enables applications that require identifier
sharing, e.g., IoT group communication.

### Privacy
A did:self DID identifier can be used for correlating the actions of a user. 






