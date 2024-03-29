[English](./README.md) | [中文](./README_zh.md)

---

# BRC-721: Non-Fungible Tokens on the Bitcoin Network

BRC-721 is inspired by the BRC-20, an experimental standard for creating fungible tokens on the Bitcoin network. BRC-20, which can be found at [brc-20-experiment](https://domo-2.gitbook.io/brc-20-experiment), aims to bring the functionality of issuing tokens to the Bitcoin ecosystem.

By building upon the ideas and principles of BRC-20, BRC-721 extends the capabilities of tokenization on the Bitcoin network to include non-fungible tokens, thus enabling a broader range of digital asset management and value representation.

BRC-721 is designed for non-fungible tokens (NFTs) on the Bitcoin network. It allows for the creation, ownership, and transfer of unique digital assets on the Bitcoin blockchain. Each token created under BRC-721 has a unique identifier, making them distinct and non-interchangeable.  

## Operations

### Deploy brc-721

Example 1, deploy NFT with an external base URI to provide metadata for each token:

``` json
{
    "p": "brc-721",
    "op": "deploy",
    "tick": "ordinals",
    "max": "10000",
    "buri": "https://ipfs.io/abc/"
}
```

* For token ID 1, the metadata is located at `https://ipfs.io/abc/1`

Example 2, deploy a brc-721 NFT and set a public key to verify the signature of the minted inscriptions:

``` json
{
    "p": "brc-721",
    "op": "deploy",
    "tick": "ordinals",
    "max": "10000",
    "sig": {
      "pk": "04E32DF42865E97135ACFB65F3BAE71BDC86F4D49150AD6A440B6F15878109880A0A2B2667F7E725CEEA70C673093BF67663E0312623C8E091B13CF2C0F11EF652",
      "fields": ["rec", "uid", "exph"]
    },
    "buri": "https://ipfs.io/abc/"
}
```

* The signature for any subsequent minted inscriptions must be signed by the private key of the sig.pk, otherwise the inscription is invalid.

Example 3, deploy NFT with onchain metadata and make the collection immutable:

``` json
{
    "p": "brc-721",
    "op": "deploy",
    "tick": "ordinals",
    "max": "10000",
    "meta": {
        "name": "Ordinals",
        "description": "Bring NFT to Bitcoin", 
        "image": "https://storage.googleapis.com/opensea-prod.appspot.com/puffs/3.png",
        "attributes": [
            {
                "trait_type": "trait1", 
                "value": "value1"
            }, ... ]
    },
    "upd": false
}
```

* All tokens of the collections will share the same metadata.

#### Inscription Specification

| Key | Required? | Description |
|---|---|---|
| p | Yes | Protocol: Helps other systems identify and process brc-721 events |
| op | Yes | Operation: Type of event (deploy, mint, update) |
| tick | Yes | Ticker: identifier of the brc-721, more than 4 letters, case insensive |
| max | Yes | Max supply: set max supply of the brc-721 |
| upd | No | Whether the metadata is updatable, default is `true` |
| buri | No | BaseURI URI for the brc-721, access `{buri}{token_id}` for the metadata of a token |
| meta | No | The metadata of the collection |
| sig | No | The public key and fields for the minted inscription's signature |

* if `upd` is `true`, the deployer can update the metadata of the collection by sending an `update` inscription; otherwise, the metadata is immutable, any `update` op will be invalid.

#### Meta Specification

| Key | Required? | Description |
|---|---|---|
| name | Yes | Identifies the asset to which this NFT represents |
| description | Yes | Describes the asset to which this NFT represents |
| image | No | A URI pointing to a resource with mime type image, or [Data URL Scheme](https://en.wikipedia.org/wiki/Data_URI_scheme), for example: `data:image/svg+xml;base64,<base64_encoded_image_bytes>` |
| attributes | No | Token attributes |

* For more information about token URI and metadata, please refer to [EIP-721](https://eips.ethereum.org/EIPS/eip-721) and [metadata standards](https://docs.opensea.io/docs/metadata-standards)

#### Sig Specification

| Key | Required? | Description |
|---|---|---|
| pk | Yes | Public key: used to verify the signature of the minted inscriptions, in Compressed/Uncompressed format, represented in hexadecimal |
| fields | No | Signature fields: `uid`, `expt`, `exph`, default value `[]` |
| rec | No | Address of the inscription recipient, if `rec` is not the recipient of the inscription, the inscription is invalid |
| uid | No | Whether the `uid` field used for the signature is unique, if provided, all subsequent minted inscriptions must have a unique `uid` field, any inscription with a repeated `uid` is invalid |
| expt | No | Whether the `expt` field used for the signature is valid, if provided, all subsequent minted inscriptions must have an `expt` field that is greater than the `block_time` of the block minting the inscription, otherwise the inscription is invalid |
| exph | No | Whether the `exph` field used for the signature is valid, if provided, all subsequent minted inscriptions must have an `exph` field that is greater than the `block_height` of the block minting the inscription, otherwise the inscription is invalid |

* If `pk` is specified, the signature for any subsequent minted inscriptions must be signed by the private key of the `pk`, otherwise the inscription is invalid

### Mint brc-721

Example 1, mint an inscription:

``` json
{
    "p": "brc-721",
    "op": "mint",
    "tick": "ordinals"
}
```

Example 2, if `sig` is specified during deployment, mint an inscription with a signature:

``` json
{
    "p": "brc-721",
    "op": "mint",
    "tick": "ordinals",
    "sig": {
      "s": "",
      "uid": "",
      "expt": ""
    }
}
```

#### Inscription Specification

| Key | Required? | Description |
|---|---|---|
| p | Yes | Protocol: Helps other systems identify and process brc-721 events |
| op | Yes | Operation: Type of event (deploy, mint, update) |
| tick | Yes | Ticker: identifier of the brc-721, 4 to 8 letters, case insensive |
| sig | No | Signature: used to verify the signature of the minted inscriptions, if `sig` is specified during deployment, the inscription must be signed |
| s | No | DER encoded signature, represented in hexadecimal |
| uid | No | The `uid` field used for the signature, used to uniquely identify the inscription, if `uid` is repeated, the inscription with the smallest inscription ID (token ID) is valid, and other inscriptions with repeated `uid` are invalid |
| expt | No | The `expt` field used for the signature, used to specify the `expt` of the inscription, if `expt` is less than the `block_time` of the block minting the inscription, the inscription is invalid |
| exph | No | The `exph` field used for the signature, used to specify the `exph` of the inscription, if `exph` is less than the `block_height` of the block minting the inscription, the inscription is invalid |

* token ID is generated from 1 to `max` according to the order of inscription IDs.

#### Signature Specification

If `sig` is specified during deployment, the inscription must be signed, the signature specification is as follows:

* If the `rec` field is specified in the `sig` of the deploy inscription, the field is the address of the recipient of the minted inscription, and does not need to be specified separately in the inscription
* Original signature message: Sort the fields specified in `fields` in lexicographical order, and then concatenate them in the format of `key=value`, for example: `exph=100&rec=xxx&uid=xxx`
  * If `key` is not in `fields`, it should not be included in the original signature message
* The original message is hashed twice using SHA256 to generate the message digest `h`
* The signature `s` is generated by signing the message digest `h` with the private key of `sig.pk` using the ECDSA secp256k1 algorithm, and encoded in DER format to generate `sig.s`

### Transfer brc-721

It's simple to transfer an brc-721 token, just send the inscription minted above to the receiver. There is no need to mint a transfer inscription before sending like brc-20.

### Update brc-721

``` json
{
    "p": "brc-721",
    "op": "update",
    "tick": "ordinals",
    "upd": false,
    "buri": "https://ipfs.io/abc/",
    "meta": { ... }
}
```

#### Inscription Specification

| Key | Required? | Description |
|---|---|---|
| p | Yes | Protocol: Helps other systems identify and process brc-721 events |
| op | Yes | Operation: Type of event (deploy, mint, update) |
| tick | Yes | Ticker: identifier of the brc-721, 4 to 8 letters, case insensive |
| upd | No | Whether the metadata is updatable, default is `true` |
| buri | No | BaseURI URI for the brc-721, access `{buri}{token_id}` for the metadata of a token |
| meta | No | The metadata of the collection |

* This operation should be only allowed for the owner of the deploy inscription
* The `update` inscription must be minted to the same address as the deploy inscription.
* Only `upd`, `buri` and `meta` can be updated.
* if `upd` is `true`, the deployer can update the metadata of the collection by sending an update inscription; otherwise, the metadata is immutable, any `update` op will be invalid after the inscription setting `upd` to `false`.

## State Changes

* NFT Deployer

  * The address holding the `deploy` inscription is the deployer
  * The receiving address of the first deploy inscription minting becomes the deployer
  * If the deploy inscription is transferred to a new address, the new address becomes the deployer

* Token ID

  * Similar to ERC721, each token in a BRC-721 collection has a unique ID
  * Each `mint` operation's inscription generates a token with a token ID ranging from 1 to `max` (the total supply defined in the deploy inscription) in the order of inscription ID
  * Minting inscriptions beyond the total supply are invalid
  * `mint` inscription ID should be larger than the deploy inscription ID

* Token Owner

  * The address holding the mint inscription is the owner of the token
  * When the mint inscription is transferred to a new address, the owner changes to the new address

* Transfer

  * Transfer the NFT token using `ord wallet send <ADDRESS> <INSCRIPTION ID>`

* Metadata

  * The metadata of the collection is immutable if `upd` is `false`
  * The metadata of the collection can be updated by the deployer if `upd` is `true`
  * The metadata of the collection is initialized by the `deploy` inscription
  * The metadata of the collection can be updated by the `update` inscription
  * The `update` inscription must be minted by the owner of `deploy` inscription and sent to the same address as the `deploy` inscription
  * `update` inscription ID should be larger than the deploy inscription ID

## FAQ

### What are the differences between brc-721 and native ordinals NFT?

BRC-721 is built upon the ordinals protocol. Although native ordinals NFT itself can store images, there are significant functional differences between brc-721 and native ordinals NFT:

* Data storage

  * Native ordinals NFT stores images for each token, which can lead to high minting fees and occupy a large amount of Bitcoin network space.

  * BRC-721 only needs to save the image once during deployment, and the mint operation does not require saving the image, which can significantly save minting fees and Bitcoin network space. Additionally, brc-721 supports storing images in off-chain services like IPFS, saving Bitcoin space and providing flexible attribute information for each token.

  * Native ordinals NFT cannot effectively index a Collection, while brc-721 provides a JSON specification similar to brc-20, enabling effective indexing and searching of NFTs within a Collection.

* BRC protocol compatibility

  * BRC-721 adopts a protocol format similar to brc-20, defining different functions through JSON content, greatly improving the flexibility of NFTs. For example, the reveal function can be implemented through the update operation; the tick field enables effective indexing of NFTs within a Collection.

* NFT ecosystem compatibility

  * ERC-721 standard NFTs are more popular in the current market. BRC-721 adopts token URI and metadata specifications consistent with ERC-721, enabling quick adaptation to the existing NFT ecosystem. Meanwhile, native ordinals do not support traits and other fields, while brc-721 supports defining NFT attributes and rarity information.

### What are the differences between brc-721 and brc-20?

* BRC-721 is used for Non-Fungible Tokens (NFT), while brc-20 is used for Fungible Tokens

* BRC-721 follows the brc-20 specifications, using JSON to define tokens and functions

* BRC-20 requires minting an inscription before transferring, which is necessary for fungible tokens, but it leads to high transfer costs and increases invalid information on the Bitcoin network. BRC-721 takes advantage of the ordinals inscription feature and can complete the transfer directly by sending, significantly reducing costs and decreasing invalid information on the network.

### What are the differences between brc-721 and brc721.com?

[brc721.com](https://www.brc721.com/) also proposes a Non-Fungible Token (NFT) solution based on the ordinals protocol, but its protocol is more complex and is not compatible with brc-20. BRC-721, on the other hand, is consistent with the brc-20 standard.

## Contribution

BRC-721 is an experimental standard that brings non-fungible tokens (NFTs) to the Bitcoin network. With this standard, users can create, mint, transfer, and update unique digital assets, enabling a wide range of use cases, such as digital art, collectibles, virtual goods, and more.  

The standard allows for a series of operations that facilitate the management of non-fungible tokens, including deployment, minting, transferring, and updating metadata. Each token is assigned a unique identifier, ensuring that each NFT is distinct and cannot be exchanged on a one-to-one basis with another NFT.  

As an experimental standard, BRC-721 invites improvements and modifications to enhance its functionality and adapt it to the growing needs of the NFT ecosystem.

## Change Log

### v0.0.3

* Add `sig` field to the deploy inscription and mint inscription

### v0.0.2

* Add `upd` field to the deploy inscription and update inscription

### v0.0.1

* Initial version
