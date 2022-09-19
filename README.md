# did:self method specification
## Author
* [Nikos Fotiou](https://www.fotiou.gr), [MMLab/AUEB](https://mm.aueb.gr), [ExcID](https://www.excid.io)

## About
did:self is a DID method that enables DID document management without registries. 
A did:self identifier is the thumbprint of a JWK as defined in [RFC 7638](https://www.rfc-editor.org/rfc/rfc7638).
The corresponding  DID document is protected by a "proof", which is a JSON Web Signature generated
by the private key that corresponds to the did:self identifier.

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
`document proof`. A `document proof` is a compact encoded 
[JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515).

The header of the proof incudes two fields:

* `alg` The algorithm used for generating the proof.
* `jwk` The JWK that can be used for verifying the proof. The thumbprint of this key **must** match the did:self identifier 

The payload of the proof is a JSON string that includes at least the following 
fields: 

* `iat` The NumericDate (as defined in Section 2 of [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519#section-2)) of 
the time at which the proof was generated.
* `exp` The NumericDate (as defined in Section 2 of [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519#section-2)) of 
the time at which the proof expires.
* `s256` The base64url encoded hash of the DID document, calculated using SHA-256.

## DID document validation
Given a did:self DID, a DID document and a `proof` any entity can attest whether
 or not the DID document is a 
valid document for the given DID using the following protocol.

1. Verify that the `s256` field of the payload of the `proof`contains 
the hash of the DID document.
1. Verify that the thumbprint of the `jwk` field of the header of `proof` is equal to the DID.
1. Verify that the current time is between the time defined by the `iat` and `exp` fields for the `proof'.
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
The proof of the DID document is signed using the using the 
private key that corresponds to the did:self DID.

The following is a valid DID document

```JSON
{
  "id": "did:self:3rdYsl79x51rfk8zMgQN7-1sStIro9cs0iUfNAqeElI",
  "authentication": [
    {
      "id": "#key1",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "kty": "EC",
        "crv": "P-256",
        "x": "5y9L_pOEyepZBP3HCcn0u7wFkTwFIL1qUUq-oFsRNJk",
        "y": "lxDZvayjRUH4r1HghIg0ZoknlWyqaATwsWtJazcUCRw"
      }
    }
  ]
}
```

The following is the corresponding proof

```
eyJhbGciOiAiRVMyNTYiLCAiandrIjogeyJrdHkiOiAiRUMiLCAiY3J2IjogIlAtMjU2IiwgIngiOiAiNXk5TF9wT0V5ZXBaQlAzSENjbjB1N3dGa1R3RklMMXFVVXEtb0ZzUk5KayIsICJ5IjogImx4RFp2YXlqUlVINHIxSGdoSWcwWm9rbmxXeXFhQVR3c1d0SmF6Y1VDUncifX0.eyJpYXQiOiAxNjYyNzM0OTU1LCAiZXhwIjogMTY2MjczNjE1NSwgInMyNTYiOiAiMVN4TEZJSkNtaFg5UVlXQXZpQjV2UWhLeFhrMGFwSWVGU3FEY2tJUjZuYyJ9.C1NUMIRFJskWoHs1v1i99Ni1YLKQ26NK2EKaRCX-J8jbYRpAsuqk2RWVPkC6xIH19Q0pSkIqlpzx8Z5t7jcntw
```

The following is the decoded proof header

```JSON
{
  "alg": "ES256",
  "jwk": {
    "kty": "EC",
    "crv": "P-256",
    "x": "5y9L_pOEyepZBP3HCcn0u7wFkTwFIL1qUUq-oFsRNJk",
    "y": "lxDZvayjRUH4r1HghIg0ZoknlWyqaATwsWtJazcUCRw"
  }
}
```

The following is the decoded proof payload

```JSON
{
  "iat": 1662734955,
  "exp": 1662736155,
  "s256": "1SxLFIJCmhX9QYWAviB5vQhKxXk0apIeFSqDckIR6nc"
}
```

The DID document may include an arbitrary number of verification methods and relationships.
The public key(s) used as verification methods do not have to be the same as the
DID identifier, although using the same key is allowed. 

### Read
The Read operation should output the DID document, 
the corresponding `document proof`. The DID document should be then validated using the process
described in [DID document validation](#did-document-validation)


### Update
With the update operation, the DID document is replaced
with a new one. A new `document proof` is also generated. 

### Deactivate
Since there is no registry the deactivate method is not supported. Nevertheless,
each DID document proof includes an expiration time and a DID document is not
valid beyond this time. 

## Security and Privacy Considerations

### Security
This method considers two types of cryptographic keys: the key that corresponds to the 
identifier, which is used for generating the DID document proofs, and the keys used
as verification methods. Although the same key can be used in both cases,
it is recommended to use different keys so that verification keys can be rotated.

A breached verification method key cannot be revoked. For this reason it is recommended
to use short-lived proofs. Nevertheless, by leveraging key identifiers and the `iat`
field of the proof a verifier can be hinted that a key in a DID document "replaces"
a key included in an older, non-expired DID document. In particular, the following
rule can be followed: if there are two DID documents, for the same DID, that include
a verification method with the same `id` then the verification method included in
the **newest** DID document replaces the verification method included in the older
DID document.  

This method  does not provide an 1:1 relationship between a did:self DID identifier and
the corresponding DID document: there can be multiple valid DID documents for the same
identifier. This is a design choice that enables applications that require identifier
sharing, e.g., IoT group communication.

### Privacy
A did:self DID identifier can be used for correlating the actions of a user. 






